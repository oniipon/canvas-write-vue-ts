<template>
	<div class="canvas_write">
		<div class="canvas">
			<canvas width="800" height="800" id="back_img"></canvas>
			<canvas id="flont_img" width='800' height='800' style=' border:2px solid #000;'></canvas>
		</div>
		<div>
			<select v-model="color" @change="change_pencil">
				<option v-for="color in colors" v-bind:key="color.code"
				        :value="color.code">
					{{color.color_name}}
				</option>
			</select>
			<button @click='change_eraser'>消しゴム</button>
			<button @click="prev_draw_img()">前に戻る</button>
			<button @click="next_draw_img()">次に進む</button>
			<button @click="all_delete()">全消し</button>
		</div>
	</div>
</template>

<script lang='ts'>
    import {Component, Prop, Vue} from "vue-property-decorator";

    interface Log {
        png: string
    }

    @Component
    export default class CanvasWrite extends Vue {
        private x = 0;
        private y = 0;
        private canvas: HTMLCanvasElement | null = null;
        private canvas_content: CanvasRenderingContext2D | null = null;
        private move_flag = false;
        private size = 12;
        private color = "#000000";
        private logs: Log[] = [];
        private log_name = "canvas_name";
        private now_index = -1;
        private colors = [
            {color_name: "黒色", code: "#000000"}
            , {color_name: "赤色", code: "#ff0000"}
            , {color_name: "黄色", code: "#ffff00"}];
        private img_path = "http://localhost:8081/img/shima.2caeef91.png";

        mouse_down(e: MouseEvent) {
            if (this.canvas == null || this.canvas_content == null) return;
            e.preventDefault();
            this.canvas_content.beginPath();
            this.x = e.layerX;
            this.y = e.layerY;
            this.canvas_content.moveTo(this.x, this.y);
        };

        mouse_move(e: MouseEvent) {
            if (this.canvas == null || this.canvas_content == null) return;
            if (e.buttons === 1) {
                this.x = e.layerX;
                this.y = e.layerY;
                this.move_flag = true;
                this.canvas_content.lineTo(this.x, this.y);
                this.canvas_content.lineCap = "round";
                this.canvas_content.lineWidth = this.size;
                this.canvas_content.strokeStyle = this.color;
                this.canvas_content.stroke();
            }
        };

        mouse_up(e: MouseEvent) {
            if (this.canvas == null || this.canvas_content == null) return;
            if (!this.move_flag) {
                this.canvas_content.lineTo(this.x - 1, this.y - 1);
                this.canvas_content.lineCap = "round";
                this.canvas_content.lineWidth = this.size;
                this.canvas_content.strokeStyle = this.color;
                this.canvas_content.stroke();
            }
            this.move_flag = false;
            this.save_log();
        };

        mounted() {
            this.canvas = document.querySelector("#flont_img");
            if (this.canvas == null) return;
            this.canvas_content = this.canvas.getContext("2d"); // canvasの種類指定
            this.canvas.addEventListener("mousedown", this.mouse_down);
            this.canvas.addEventListener("mousemove", this.mouse_move);
            this.canvas.addEventListener("mouseup", this.mouse_up);
            this.set_back_img();
            this.canvas_content.globalAlpha = 1;
        }

        set_back_img() {
            const canvas: HTMLCanvasElement = document.querySelector("#back_img");
            const canvas_content = canvas.getContext("2d");
            const img = new Image();
            img.src = this.img_path;
            img.onload = () => canvas_content.drawImage(img, 0, 0, 800, 800);
        }

        save_log() {
            if (this.canvas == null || this.canvas_content == null) return;
            const png = this.canvas.toDataURL();
            this.logs = this.logs.filter((d, i) => i <= this.now_index);
            this.logs.push({png});
            this.now_index = this.logs.length - 1;
        }

        prev_draw_img() {
            if (this.now_index === -1) return;
            this.now_index--;
            if (this.now_index === -1) {
                this.reset_canvas();
                return;
            }
            this.log_render();
        }

        next_draw_img() {
            if (this.logs.length <= this.now_index + 1) return;
            this.now_index++;
            this.log_render();
        }

        log_render() {
            this.reset_canvas();
            const erase_flag = this.canvas_content.globalCompositeOperation === "destination-out" ? true : false;
            this.change_pencil(); // 消しゴムモードだとレンダリングが出来ないっぽい
            this.draw(this.logs[this.now_index].png, erase_flag);
        }

        reset_canvas() {
            if (this.canvas_content == null) return;
            this.canvas_content.clearRect(0, 0, this.canvas_content.canvas.clientWidth, this.canvas_content.canvas.clientHeight);
        }

        draw(src: string, erase_flag: boolean) {
            if (this.canvas_content == null) return;
            const img = new Image();
            img.src = src;
            const canvas_content = this.canvas_content;
            img.onload = () => {
                canvas_content.drawImage(img, 0, 0);
                if (erase_flag) this.change_eraser(); // 消しゴムモードに戻す
            };
        }

        change_eraser() {
            this.canvas_content.globalCompositeOperation = "destination-out";
        }

        change_pencil() {
            this.canvas_content.globalCompositeOperation = "source-over";
        }

        all_delete() {
            this.reset_canvas();
            this.now_index = -1;
            this.logs = [];
        }
    }
</script>

<style scoped>
	.canvas {
		position: relative;
		margin: 0 auto;
		top: 30px;
		right: 400px;
	}

	canvas {
		position: absolute;
	}

	.canvas_write {
		display: flex;
		flex-direction: column;
	}
</style>