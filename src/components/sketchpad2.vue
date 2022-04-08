<template>
  <div class="wraper" ref="wraper">
    <div class="canvas-wraper">
      <canvas id="canvas" ref="canvas" width="1000" height="1000"></canvas>
      <img id="test-img" src="../assets/img/cup.png" style="display: none">
    </div>
  </div>
</template>
<script>
import { fabric } from 'fabric';
export default {
  data () {
    return {

    }
  },
  computed: {
    canvasWidth () {
      return window.innerWidth
    }
  },
  created () {
    // console.log('create');
  },
  mounted () {
    this.initCanvas()
  },
  methods: {
    initCanvas () {
      let _this = this;
      let canvas = this.__canvas = new fabric.Canvas('canvas');
      let img = document.getElementById('test-img');
      canvas.freeDrawingBrush.color = '#E34F51'
      canvas.freeDrawingBrush.width = 2;
      fabric.Object.prototype.transparentCorners = false;
      var radius = 300;
      canvas.preserveObjectStacking = true;

      //   let img_url = require('../assets/img/cup.png');
      //   let image = new Image();
      //   image.src = img_url;
      //   canvas.setBackgroundImage(require('../assets/img/cup.png'), canvas.renderAll.bind(canvas), {
      //     opacity: 1,
      //     width: image.width,
      //     height: image.height,
      //     repeat: 'no-repeat',
      //     // Needed to position backgroundImage at 0/0
      //     originX: 'left',
      //     originY: 'top',
      //   });



      fabric.Image.fromURL(require('../assets/img/cup.png'), function (img) {
        var scalar = 1, abort;
        var path = 'M 500 200 L 500 250 L 350 250 L 350 200 Z M 500 300 L 500 350 L 350 350 L 350 300 Z';
        var shell = new fabric.Path(path, {
          fill: '',
          stroke: 'blue',
          strokeWidth: 5,
          scaleX: 2,
          scaleY: 2,
          lockScalingX: true,
          lockScalingY: true,
          lockSkewingX: true,
          lockSkewingY: true,
          originX: 'center',
          originY: 'center',
        });
        var clipPath = new fabric.Path(path, {
          absolutePositioned: true,
          originX: 'center',
          originY: 'center',
          scaleX: 2,
          scaleY: 2
        });

        img.scale(0.5).set({
          left: 200,
          top: 180,
          clipPath: clipPath
        });
        shell.on('moving', ({ e, transform, pointer }) => {
          //  only because they are absolutePositioned
          clipPath.setPositionByOrigin(shell.getCenterPoint(), 'center', 'center');
          img.set('dirty', true);
        });
        shell.on('rotating', () => {
          clipPath.set({ angle: shell.angle });
          img.set('dirty', true);
        });
        shell.on('selected', () => {
          //   abort();
        });
        shell.on('deselected', () => {
          scalar = 1;
        });


        img.clipPath = clipPath;
        canvas.add(img, shell);
        canvas.setActiveObject(img);
      });
    },
  }
}
</script>

<!--<style lang="scss" scoped>-->
<style scoped>
.wraper {
  position: relative;
  width: 100%;
  height: 100%;
}
.wraper .canvas-wraper {
  height: 50%;
  width: 100%;
  margin-bottom: 10px;
  overflow: hidden;
}
.wraper .controlPanel {
  width: 100%;
  height: 62px;
  /* background: #ddd; */
  display: flex;
  justify-content: flex-start;
  align-items: center;
  margin-bottom: 15px;
}
.wraper .controlPanel .contro-item {
  flex-basis: 100px;
  border-right: 1px solid #dad7d9;
  text-align: center;
  cursor: pointer;
  background: #fefefe;
}
.wraper .controlPanel .contro-item i {
  font-size: 38px;
  line-height: 62px;
}
.wraper .controlPanel .contro-item.active i {
  background: #e34f51;
  color: #fff;
  border-radius: 3px;
  font-size: 42px;
}
.download img {
  width: 100%;
}
.box_dialog {
  padding: 20px;
}
.wraper >>> .el-dialog__header {
  padding: 5px 20px;
  text-align: center;
}
.box_cm {
  margin-left: 10px;
}
.box_dialog >>> .dialog-footer {
  display: flex;
  flex-direction: row;
  justify-content: flex-end;
}
</style>