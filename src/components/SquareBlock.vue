  <template>  


  <div class="square-block-container">
  <canvas ref="lineCanvas" style="position: absolute; top: 7%; left: 360px; width: 400px; height: 160px; "></canvas>

      <div class="square-block" ref="square1"
          @mousedown="startMoving"
          @mouseup="stopMoving"
          @mousemove="moveSquare1"
          @touchstart="startMoving"
          @touchend="stopMoving"
          @touchmove="moveSquare1"
          :style="{ left: x1 + 'px', top: y1 + 'px', transform: 'scale(' + scale + ')' }"
          style="width: 300px; height: 180px; border: 1px solid black; display: inline-block; position: relative;">
          
        <div class="inner-block" style="width: 100%; height: 33.33%; border-bottom: 1px solid black; float: left;"></div>
        <div class="inner-block" style="width: 100%; height: 33.33%; border-bottom: 1px solid black; float: left;"></div>
        <div class="inner-block" style="width: 100%; height: 33.33%; float: left;"></div>
        <button class="top-left-button" style="position: absolute; top: 0; left: -6px; width: 20px; height: 20px; background-color: lightgray; border-radius:50%;"></button>
        <button  class="bottom-right-button" style="position: absolute; bottom: 40px; right: -6px; width: 15px; height: 15px; background-color: lightgray;border-radius:50%;"></button>
        
      </div>
      
      
      <div class="square-block" ref="square2"
          @mousedown="startMoving"
          @mouseup="stopMoving"
          @mousemove="moveSquare2"
          @touchstart="startMoving"
          @touchend="stopMoving"
          @touchmove="moveSquare2"
          :style="{ left: x2 + 'px', top: y2 + 'px', transform: 'scale(' + scale + ')' }"
          style="width: 300px; height: 180px; border: 1px solid black; display: inline-block; position: relative; margin-left: 400px;">
        <div class="inner-block" style="width: 100%; height: 33.33%; border-bottom: 1px solid black; float: left;"></div>
        <div class="inner-block" style="width: 100%; height: 33.33%; border-bottom: 1px solid black; float: left;"></div>
        <div class="inner-block" style="width: 100%; height: 33.33%; float: left;"></div>
        <button class="top-left-button" style="position: absolute; top: 30px; left: -6px; width: 20px; height: 20px; background-color: lightgray;border-radius:50%;"></button>
        <button class="bottom-right-button" style="position: absolute; bottom: 0; right: -6px; width: 15px; height: 15px; background-color: lightgray;border-radius:50%;"></button>
      </div>
      <div class="zoom-buttons" style="position: relative; top: 100px; ">
        <button @click="zoomIn" >+</button>
        <button @click="zoomOut" >-</button>
        <button @click="undo">Undo</button>
        <button @click="redo">Redo</button>  
      </div>
      
      
      <canvas style="width: 100%; height: 100%; margin-top:-30%;  background-color:#F5F5F5	 "></canvas>
  </div>
  </template>
  <script>
  import * as d3 from "d3";


  export default {
    data() {
      return {
        x1: 50,
        y1: 50,
        x2: 50,
        y2: 50,
        x3: 50,
        y3: 50,
        x4: 50,
        y4:50,
        moving: false,
        currentSquare: null,
        scale: 1,
        history: [],
        future: [],
        path:null,
        lineCoordinates:[0, 0, 0, 0],
        
        
      };
      
    },
    mounted() {
    const canvas = this.$refs.lineCanvas;
    const ctx = canvas.getContext("2d");
    const button = this.$refs.square1.querySelector('.bottom-right-button');
    button.addEventListener('click', () => {
    ctx.beginPath();
    ctx.moveTo(0, 125);
    ctx.lineTo(400, 10);
    ctx.strokeStyle = 'red';
    ctx.stroke();
  });
  },
  
  
    methods: {

  startMoving(event) {
    this.moving = true;
      if (event.target.classList.contains("top-left-button")) {
          this.currentSquare = "square1";
      } else if (event.target.classList.contains("bottom-right-button")) {
          this.currentSquare = "square2";
        }
  },

  stopMoving() {
        this.moving = false;
  },
     
  moveSquare1(event) {
    if (!this.moving) return;
    this.history.push({ x1: this.x1, y1: this.y1, x2: this.x2, y2: this.y2, scale: this.scale });
    this.x1 += event.movementX;
    this.y1 += event.movementY;
    this.drawLine();
    
  
  
  },
  moveSquare2(event) {
    if (!this.moving) return;
    this.history.push({ x1: this.x1, y1: this.y1, x2: this.x2, y2: this.y2, scale: this.scale });
    this.x2 += event.movementX;
    this.y2 += event.movementY;
    this.drawLine();  
  },

  zoomIn() {
    this.history.push({ x1: this.x1, y1: this.y1, x2: this.x2, y2: this.y2, scale: this.scale });
    this.scale += 0.1;
  },

  zoomOut() {
    this.history.push({ x1: this.x1, y1: this.y1, x2: this.x2, y2: this.y2, scale: this.scale });
    this.scale -= 0.1; 
  },

  undo() {
    if (this.history.length > 0) {
      const previousState = this.history.pop();
      this.future.push({ x1: this.x1, y1: this.y1, x2: this.x2, y2: this.y2, scale: this.scale });
      this.x1 = previousState.x1;
      this.y1 = previousState.y1;
      this.x2 = previousState.x2;
      this.y2 = previousState.y2;
      this.scale = previousState.scale;
      this.$ref.lineCanvas = previousState.scale;
        }
  },

  redo() {
    if (this.future.length = 0) {
      const nextState = this.future.pop();
      this.history.push({ x1: this.x1, y1: this.y1, x2: this.x2, y2: this.y2, scale: this.scale });
      this.x1 = nextState.x1;
      this.y1 = nextState.y1;
      this.x2 = nextState.x2;
      this.y2 = nextState.y2;
      this.scale = nextState.scale;
      this.$ref.lineCanvas = nextState.scale;
        }
  },
  drawLine() {
    const ctx = canvas.getContext("2d");
    const rect1 = this.$refs.square1.getBoundingClientRect();
    const rect2 = this.$refs.square2.getBoundingClientRect();   
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.beginPath();
    ctx.moveTo(0 , 125);
    ctx.lineTo(400, 10);
    ctx.stroke();
   },
 
  }
  };
  </script>
  <style scoped>
  .square-block {
    cursor: grab;
  }

  .top-left-button:active,
  .bottom-right-button:active {
    cursor: grabbing;
  }

  </style>


