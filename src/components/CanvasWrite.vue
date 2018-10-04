<template>
	<div>
		<canvas width='1000' height='1000' style=' border:2px solid #000;'></canvas>
		<div>
			<button onclick=''>消しゴム</button>
			<button @click="prev_draw_img()">前に戻る</button>
      <button @click="next_draw_img()">次に進む</button>
			<button>全消し</button>
		</div>
	</div>
</template>

<script lang='ts'>
    import {Component, Prop, Vue} from "vue-property-decorator";
interface Log{
  png:string
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
        private logs:Log[] = [];
        private log_name = "canvas_name";
        private now_index = -1;

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
            this.canvas = document.querySelector("canvas");
            if (this.canvas == null) return;
            this.canvas_content = this.canvas.getContext("2d"); // canvasの種類指定
            this.canvas.addEventListener("mousedown", this.mouse_down);
            this.canvas.addEventListener("mousemove", this.mouse_move);
            this.canvas.addEventListener("mouseup", this.mouse_up);
        }

        save_log() {
          if (this.canvas == null || this.canvas_content == null) return;
          const png = this.canvas.toDataURL();
          this.logs.push({png});
          this.now_index = this.logs.length-1;
        }

        prev_draw_img(){
          if(this.now_index ===-1) return
          this.now_index--;
          if(this.now_index === -1){
            this.reset_canvas()
            return
          }
           this.reset_canvas()
          this.draw(this.logs[this.now_index].png);
        }
        next_draw_img(){
          if(this.logs.length<=this.now_index+1) return;
              this.now_index++;
             this.reset_canvas()
            console.log(this.now_index);
            this.draw(this.logs[this.now_index].png);
        }

        reset_canvas(){
          if(this.canvas_content == null) return;
          this.canvas_content.clearRect(0, 0, this.canvas_content.canvas.clientWidth, this.canvas_content.canvas.clientHeight);
        }

        draw(src:string) {
          if(this.canvas_content == null) return;
          console.log('draw');
          const img = new Image();
          img.src = src;
          const canvas_content = this.canvas_content;
          img.onload = ()=>canvas_content.drawImage(img,0,0);
        }
    }
</script>

<style scoped>
</style>