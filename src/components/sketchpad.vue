<template>
  <div class="wraper" ref="wraper">
    <div class="canvas-wraper">
      <canvas id="canvas" ref="canvas"></canvas>
      <img id="test-img" src="../assets/img/kotei_9628.png" style="display: none">
    </div>
    <div class="controlPanel">
      <div :class="[initIdx===idx ? 'contro-item active' : 'contro-item']" v-for="(item,idx) in toolsArr" :key="idx" @click="handleTools(item, idx)">
        <i :class="'iconfont' + item.icon"></i>
      </div>
    </div>
    <div class="download">
      <button type="button" :disabled="done" @click="downLoadImage">转换为base64并预览</button>
      <img :src="imageBase64" v-show="imageBase64!=''" alt="">
    </div>
    <el-dialog title="尺度标准" :visible.sync="dialogVisible" width="30%" :before-close="handleClose">
      <div class="box_dialog">
        <div>
          <el-input-number v-model="num" @change="handleChange" :step="1" :min="0.01" :precision="2" label="请输入尺度"></el-input-number>
          <span class="box_cm">cm</span>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="handleClose">取 消</el-button>
          <el-button type="primary" @click="handleSure">确 定</el-button>
        </span>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import { fabric } from 'fabric'
export default {
  data () {
    return {
      dialogVisible: false, // 弹窗
      num: 1, // 标准尺寸
      lineWidth: 0, // 标准线段在页面上显示的长度
      lineWidthUnit: 0, // 单位标准线段的长度，1px代表 多少 cm
      currentTool: '',
      done: false,
      fabricObj: null,
      initIdx: 0,
      toolsArr: [
        { name: 'pencil', icon: ' icon-pencil' },
        { name: 'line', icon: ' icon-line' },
        { name: 'arrow', icon: ' icon-arrow' },
        { name: 'xuxian', icon: ' icon-xuxian' },
        { name: 'text', icon: ' icon-ziti' },
        { name: 'juxing', icon: ' icon-juxing' },
        { name: 'cricle', icon: ' icon-yuanxing' },
        { name: 'ellipse', icon: ' icon-tuoyuanxing' },
        { name: 'equilateral', icon: ' icon-sanjiaoxing' },
        { name: 'remove', icon: ' icon-remove' },
        { name: 'undo', icon: ' icon-huitui' },
        { name: 'redo', icon: ' icon-xiangqian' },
        { name: 'reset', icon: ' icon-reset' },
        { name: 'move', icon: ' el-icon-mouse' },
      ],
      mouseFrom: {},
      mouseTo: {},
      moveCount: 1,
      doDrawing: false,
      fabricHistoryJson: [],
      mods: 0,
      drawingObject: null, //绘制对象
      drawColor: '#E34F51',
      drawWidth: 2,
      imageBase64: '',
      zoom: window.zoom ? window.zoom : 1,
    }
  },
  computed: {
    canvasWidth () {
      return window.innerWidth
    }
  },
  created () {
    console.log('create');
  },
  mounted () {
    console.log('mounted');
    //初始化canvas
    this.initCanvas()
  },
  methods: {
    initCanvas () {
      let _this = this
      this.fabricObj = new fabric.Canvas('canvas', {
        width: this.canvasWidth - 50,
        height: 500,
        // isDrawingMode: false,
        // selectable: true,
        // selection: false,
        devicePixelRatio: true, //Retina 高清屏 屏幕支持
      });
      let img = document.getElementById('test-img');
      this.fabricObj.freeDrawingBrush.color = '#E34F51'
      this.fabricObj.freeDrawingBrush.width = 2

      this.fabricObj.add(
        // new fabric.Rect({ top: 100, left: 100, width: 500, height: 100, fill: '#f55' }),
        // new fabric.Circle({ top: 140, left: 230, radius: 75, fill: 'green' }),
        // new fabric.Triangle({ top: 300, left: 210, width: 100, height: 100, fill: 'blue' }),
        // new fabric.Image(img, { top: 0, left: 0, width: 500, hegith: 500 }),
      );
      // let imgInstance = new fabric.Image(img, {
      //   left: 100, // 图片相对画布的左侧距离
      //   top: 100, // 图片相对画布的顶部距离
      //   width: 100,
      //   height: 100,
      //   angle: 30, // 图片旋转角度
      //   opacity: 0.85, // 图片透明度
      //   // 这里可以通过scaleX和scaleY来设置图片绘制后的大小，这里为原来大小的一半
      //   // scaleX: 0.5, 
      //   // scaleY: 0.5
      // });
      // imgInstance.set({
      //   borderColor: 'red',
      //   cornerColor: 'green',
      //   cornerSize: 6
      // });
      // this.fabricObj.add(imgInstance);

      var yellow = new fabric.Circle({
        top: 200,
        left: 0,
        radius: 100,
        strokeDashArray: [5, 5],
        stroke: 'black',
        strokeWidth: 5,
        fill: 'yellow'
      });
      this.fabricObj.add(yellow);

      var blue = new fabric.Circle({
        top: 150,
        left: 80,
        radius: 100,
        strokeDashArray: [5, 5],
        stroke: 'black',
        strokeWidth: 5,
        fill: 'blue',
        globalCompositeOperation: 'source-atop'
      });
      this.fabricObj.add(blue);

      var green = new fabric.Circle({
        top: 0,
        left: 0,
        radius: 100,
        strokeDashArray: [5, 5],
        stroke: 'black',
        strokeWidth: 5,
        fill: 'green'
      });
      this.fabricObj.add(green);

      //绑定画板事件
      this.fabricObjAddEvent();
    },
    //时间监听
    fabricObjAddEvent () {
      this.panning = false;
      this.fabricObj.on({
        'mouse:down': (o) => {
          this.mouseFrom.x = o.pointer.x;
          this.mouseFrom.y = o.pointer.y;
          this.doDrawing = true;
          if (this.currentTool == 'text') {
            this.drawText()
          }
        },
        'mouse:up': (o) => {
          this.mouseTo.x = o.pointer.x;
          this.mouseTo.y = o.pointer.y;
          this.drawingObject = null;
          this.moveCount = 1;
          this.doDrawing = false;
          this.updateModifications(true);
          if (this.currentTool == 'line') {
            this.dialogVisible = true;
          }
          if (this.currentTool == 'juxing') {
            this.drawTextUnit()
          }
        },
        'mouse:move': (o) => {
          if (this.moveCount % 2 && !this.doDrawing) {
            //减少绘制频率
            return;
          }
          this.moveCount++;
          this.mouseTo.x = o.pointer.x;
          this.mouseTo.y = o.pointer.y;;
          this.drawing();
        },
        //对象移动时间 限制移动在画布内
        'object:moving': (e) => {
          e.target.opacity = 0.5;
          // var obj = e.target;
          // // if object is too big ignore
          // if (obj.currentHeight > obj.canvas.height || obj.currentWidth > obj.canvas.width) {
          //   return;
          // }
          // obj.setCoords();
          // // top-left  corner
          // if (obj.getBoundingRect().top < 0 || obj.getBoundingRect().left < 0) {
          //   obj.top = Math.max(obj.top, obj.top - obj.getBoundingRect().top + 20);
          //   obj.left = Math.max(obj.left, obj.left - obj.getBoundingRect().left + 20);
          // }
          // // bot-right corner
          // if (obj.getBoundingRect().top + obj.getBoundingRect().height > obj.canvas.height || obj.getBoundingRect().left + obj.getBoundingRect().width > obj.canvas.width) {
          //   obj.top = Math.min(obj.top, obj.canvas.height - obj.getBoundingRect().height + obj.top - obj.getBoundingRect().top - 20);
          //   obj.left = Math.min(obj.left, obj.canvas.width - obj.getBoundingRect().width + obj.left - obj.getBoundingRect().left - 20);
          // }
        },
        //增加对象
        'object:added': (e) => {
          // debugger
        },
        'object:modified': (e) => {
          e.target.opacity = 1;
          let object = e.target;
          this.updateModifications(true)
        },
        'selection:created': (e) => {
          if (this.currentTool !== 'move') {
            if (e.target._objects) {
              //多选删除
              var etCount = e.target._objects.length;
              for (var etindex = 0; etindex < etCount; etindex++) {
                this.fabricObj.remove(e.target._objects[etindex]);
              }
            } else {
              //单选删除
              this.fabricObj.remove(e.target);
            }
            this.fabricObj.discardActiveObject(); //清楚选中框
            this.updateModifications(true)
          }
        },
      });
    },
    //储存历史记录
    updateModifications (savehistory) {
      if (savehistory == true) {
        this.fabricHistoryJson.push(JSON.stringify(this.fabricObj))
      }
    },
    //canvas 历史后退
    undo () {
      let state = this.fabricHistoryJson
      if (this.mods < state.length) {
        this.fabricObj.clear().renderAll();
        this.fabricObj.loadFromJSON(state[state.length - 1 - this.mods - 1]);
        this.fabricObj.renderAll();
        this.mods += 1;
      }
    },
    //前进
    redo () {
      let state = this.fabricHistoryJson
      if (this.mods > 0) {
        this.fabricObj.clear().renderAll();
        this.fabricObj.loadFromJSON(state[state.length - 1 - this.mods + 1]);
        this.fabricObj.renderAll();
        this.mods -= 1;
      }
    },
    transformMouse (mouseX, mouseY) {
      return { x: mouseX / this.zoom, y: mouseY / this.zoom };
    },
    resetObj () {
      this.fabricObj.selectable = false
      this.fabricObj.selection = false
      this.fabricObj.skipTargetFind = true
      //清除文字对象
      if (this.textboxObj) {
        this.textboxObj.exitEditing();
        this.textboxObj = null;
      }
    },
    handleTools (tools, idx) {
      this.initIdx = idx;
      this.currentTool = tools.name;
      this.fabricObj.isDrawingMode = false;
      this.resetObj();

      switch (tools.name) {
        case 'pencil':
          this.fabricObj.isDrawingMode = true;
          break;
        case 'remove':
          this.fabricObj.selection = true;
          this.fabricObj.skipTargetFind = false;
          this.fabricObj.selectable = true;
          break;
        case 'reset':
          this.fabricObj.clear();
          break;
        case 'redo':
          this.redo();
          break;
        case 'undo':
          this.undo();
          break;
        case 'move':
          this.fabricObj.skipTargetFind = false;
          break;
        default:
          break;
      }
    },
    // 绘制文字对象
    drawText () {
      this.textboxObj = new fabric.Textbox(" ", {
        left: this.mouseFrom.x,
        top: this.mouseFrom.y,
        // width: 220,
        fontSize: 18,
        fill: this.drawColor,
        hasControls: true,
        globalCompositeOperation: 'source-atop'
      });
      this.fabricObj.add(this.textboxObj);
      this.textboxObj.enterEditing();
      this.textboxObj.hiddenTextarea.focus();
      this.updateModifications(true)
    },
    // 绘制文字备注
    drawTextUnit () {
      const w = Math.abs(this.mouseTo.x - this.mouseFrom.x);
      const h = Math.abs(this.mouseTo.y - this.mouseFrom.y);
      const textLeft = Math.abs((this.mouseTo.x - this.mouseFrom.x) / 2 + this.mouseFrom.x);
      const textTop = Math.abs((this.mouseTo.y - this.mouseFrom.y) / 2 + this.mouseFrom.y);
      console.log('this.lineWidthUnit', this.lineWidthUnit)
      this.textboxObjWidth = new fabric.Textbox(`${(this.lineWidthUnit * w).toFixed(2)}cm`, {
        left: textLeft - 20,
        top: this.mouseFrom.y - 20,
        fontSize: 18,
        fill: this.drawColor,
        hasControls: true,
      });
      this.fabricObj.add(this.textboxObjWidth);

      this.textboxObjHeight = new fabric.Textbox(`${(this.lineWidthUnit * h).toFixed(2)}cm`, {
        left: this.mouseTo.x + 5,
        top: textTop - 10,
        fontSize: 18,
        fill: this.drawColor,
        hasControls: true,
      });
      this.fabricObj.add(this.textboxObjHeight);

      let objs = this.fabricObj.getObjects();
      var group = new fabric.Group([this.textboxObjWidth, this.textboxObjHeight, objs[objs.length - 3]], {})

      this.fabricObj.add(group);

      console.log('objs', objs)
      // this.textboxObj.enterEditing();
      // this.textboxObj.hiddenTextarea.focus();
      // this.updateModifications(true)
    },
    drawing () {
      let _this = this;
      const w = Math.abs(this.mouseTo.x - this.mouseFrom.x);
      const h = Math.abs(this.mouseTo.y - this.mouseFrom.y);

      if (this.drawingObject) {
        this.fabricObj.remove(this.drawingObject)
      }
      let fabricObject = null
      switch (this.currentTool) {
        case 'pencil':
          this.fabricObj.isDrawingMode = true
          break;
        case 'line':
          fabricObject = new fabric.Line([this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y], {
            stroke: this.drawColor,
            strokeWidth: this.drawWidth
          });

          // 勾股定理求线段长度
          this.lineWidth = Math.sqrt(Math.pow(w, 2) + Math.pow(h, 2));
          break;
        case 'arrow':
          fabricObject = new fabric.Path(this.drawArrow(this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y, 17.5, 17.5), {
            stroke: this.drawColor,
            fill: "rgba(255,255,255,0)",
            strokeWidth: this.drawWidth
          });
          break;
        case 'xuxian': //doshed line
          fabricObject = new fabric.Line([this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y], {
            strokeDashArray: [10, 3],
            stroke: this.drawColor,
            strokeWidth: this.drawWidth
          })
          break;
        case 'juxing': //矩形
          let path = "M " +
            this.mouseFrom.x + " " +
            this.mouseFrom.y + " L " +
            this.mouseTo.x + " " +
            this.mouseFrom.y + " L " +
            this.mouseTo.x + " " +
            this.mouseTo.y + " L " +
            this.mouseFrom.x + " " +
            this.mouseTo.y + " L " +
            this.mouseFrom.x + " " +
            this.mouseFrom.y + " z";

          fabricObject = new fabric.Path(path, {
            left: this.mouseFrom.x,
            top: this.mouseFrom.y,
            stroke: this.drawColor,
            strokeWidth: this.drawWidth,
            fill: "white",
          });
          break;
        case "cricle": //正圆
          let radius = Math.sqrt((this.mouseTo.x - this.mouseFrom.x) * (this.mouseTo.x - this.mouseFrom.x) + (this.mouseTo.y - this.mouseFrom.y) * (this.mouseTo.y - this.mouseFrom.y)) / 2;
          fabricObject = new fabric.Circle({
            left: this.mouseFrom.x,
            top: this.mouseFrom.y,
            stroke: this.drawColor,
            fill: "rgba(255, 255, 255, 0)",
            radius: radius,
            strokeWidth: this.drawWidth
          });
          break;
        case "ellipse": //椭圆
          let left = this.mouseFrom.x;
          let top = this.mouseFrom.y;
          let ellipse = Math.sqrt((this.mouseTo.x - left) * (this.mouseTo.x - left) + (this.mouseTo.y - top) * (this.mouseTo.y - top)) / 2;
          fabricObject = new fabric.Ellipse({
            left: left,
            top: top,
            stroke: this.drawColor,
            fill: "rgba(255, 255, 255, 0)",
            originX: "center",
            originY: "center",
            rx: Math.abs(left - this.mouseTo.x),
            ry: Math.abs(top - this.mouseTo.y),
            strokeWidth: this.drawWidth
          });
          break;
        case "equilateral": //等边三角形
          let height = this.mouseTo.y - this.mouseFrom.y;
          fabricObject = new fabric.Triangle({
            top: this.mouseFrom.y,
            left: this.mouseFrom.x,
            width: Math.sqrt(Math.pow(height, 2) + Math.pow(height / 2.0, 2)),
            height: height,
            stroke: this.drawColor,
            strokeWidth: this.drawWidth,
            fill: "rgba(255,255,255,0)"
          });
          break;
        case 'remove':
          break;
        default:
          // statements_def'
          break;
      }
      if (fabricObject) {
        this.fabricObj.add(fabricObject)
        this.drawingObject = fabricObject;
      }
    },
    //书写箭头的方法
    drawArrow (fromX, fromY, toX, toY, theta, headlen) {
      theta = typeof theta != "undefined" ? theta : 30;
      headlen = typeof theta != "undefined" ? headlen : 10;
      // 计算各角度和对应的P2,P3坐标
      let angle = Math.atan2(fromY - toY, fromX - toX) * 180 / Math.PI,
        angle1 = (angle + theta) * Math.PI / 180,
        angle2 = (angle - theta) * Math.PI / 180,
        topX = headlen * Math.cos(angle1),
        topY = headlen * Math.sin(angle1),
        botX = headlen * Math.cos(angle2),
        botY = headlen * Math.sin(angle2);
      let arrowX = fromX - topX,
        arrowY = fromY - topY;
      let path = " M " + fromX + " " + fromY;
      path += " L " + toX + " " + toY;
      arrowX = toX + topX;
      arrowY = toY + topY;
      path += " M " + arrowX + " " + arrowY;
      path += " L " + toX + " " + toY;
      arrowX = toX + botX;
      arrowY = toY + botY;
      path += " L " + arrowX + " " + arrowY;
      return path;
    },
    downLoadImage () {
      this.done = true
      //生成双倍像素比的图片
      let base64URl = this.fabricObj.toDataURL({
        formart: 'png',
        multiplier: 2
      })
      this.imageBase64 = base64URl
      this.done = false
    },
    /* 关闭弹窗 */
    handleClose () {
      this.dialogVisible = false;
      this.undo();
    },
    /* 获取数字框值 */
    handleChange (value) {
      this.num = value;
    },
    /* 确认弹窗，计算标准 */
    handleSure () {
      this.lineWidthUnit = this.num / this.lineWidth;
      this.dialogVisible = false;
      // this.undo();
      console.log('lineWidthUnit', this.lineWidthUnit);
    }
  },
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
  background: #ddd;
  display: flex;
  justify-content: center;
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