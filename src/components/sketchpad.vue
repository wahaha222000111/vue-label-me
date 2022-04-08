<template>
  <div class="wraper" ref="wraper">
    <div class="canvas-wraper">
      <canvas id="canvas" ref="canvas"></canvas>
      <img id="test-img" src="../assets/img/cup.png" style="display: none">
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
import { fabric } from 'fabric';
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
        // { name: 'pencil', icon: ' icon-pencil' },
        { name: 'move', icon: ' el-icon-mouse' },
        { name: 'line', icon: ' icon-line' },
        { name: 'picture', icon: ' el-icon-picture-outline' },
        // { name: 'arrow', icon: ' icon-arrow' },
        // { name: 'xuxian', icon: ' icon-xuxian' },
        { name: 'text', icon: ' icon-ziti' },
        { name: 'juxing', icon: ' icon-juxing' },
        { name: 'circle', icon: ' icon-yuanxing' },
        { name: 'ellipse', icon: ' icon-tuoyuanxing' },
        // { name: 'equilateral', icon: ' icon-sanjiaoxing' },
        { name: 'remove', icon: ' icon-remove' },
        // { name: 'undo', icon: ' icon-huitui' },
        // { name: 'redo', icon: ' icon-xiangqian' },
        { name: 'reset', icon: ' icon-reset' },
        { name: 'down', icon: ' el-icon-download' },
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
    // console.log('create');
  },
  mounted () {
    // console.log('mounted');
    //初始化canvas
    this.initCanvas()
  },
  methods: {
    initCanvas () {
      let _this = this
      this.fabricObj = new fabric.Canvas('canvas', {
        width: this.canvasWidth - 50,
        height: 500,
        // backgroundColor: 'black',
        // isDrawingMode: false,
        // selectable: true,
        // selection: false,
        devicePixelRatio: true, //Retina 高清屏 屏幕支持
      });
      let img = document.getElementById('test-img');
      this.fabricObj.freeDrawingBrush.color = '#E34F51'
      this.fabricObj.freeDrawingBrush.width = 2

      this.fabricObj.add(
        // new fabric.Rect({ top: 100, left: 100, width: 500, height: 100, fill: 'rgba(255,255,255,0)', stroke: '#000' }),
        // new fabric.Circle({ top: 140, left: 230, radius: 75, fill: 'green' }),
        // new fabric.Triangle({ top: 300, left: 210, width: 100, height: 100, fill: 'blue' }),
        // new fabric.Image(img, { top: 0, left: 0, width: 500, hegith: 500 }),
      );

      fabric.Object.prototype.transparentCorners = false;
      this.fabricObj.preserveObjectStacking = true;

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
          if (this.currentTool == 'juxing' || this.currentTool == 'ellipse' || this.currentTool == 'circle') {
            this.drawTextUnit();
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
          let padding = 10;
          var obj = e.target;

          // if object is too big ignore
          if (obj.currentHeight > obj.canvas.height - padding * 2 ||
            obj.currentWidth > obj.canvas.width - padding * 2) {
            return;
          }
          obj.setCoords();

          // top-left corner
          if (obj.getBoundingRect().top < padding ||
            obj.getBoundingRect().left < padding) {
            obj.top = Math.max(obj.top, obj.top - obj.getBoundingRect().top + padding);
            obj.left = Math.max(obj.left, obj.left - obj.getBoundingRect().left + padding);
          }

          // bot-right corner
          if (obj.getBoundingRect().top + obj.getBoundingRect().height > obj.canvas.height - padding ||
            obj.getBoundingRect().left + obj.getBoundingRect().width > obj.canvas.width - padding) {
            obj.top = Math.min(
              obj.top,
              obj.canvas.height - obj.getBoundingRect().height + obj.top - obj.getBoundingRect().top - padding);
            obj.left = Math.min(
              obj.left,
              obj.canvas.width - obj.getBoundingRect().width + obj.left - obj.getBoundingRect().left - padding);
          }
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
        case 'picture':
          this.renderPicture();
          break;
        case 'down':
          this.handleDown();
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
        // globalCompositeOperation: 'source-atop'
      });
      this.fabricObj.add(this.textboxObj);
      this.textboxObj.enterEditing();
      this.textboxObj.hiddenTextarea.focus();
      this.updateModifications(true)
    },
    // 绘制文字备注
    drawTextUnit () {
      console.log('this.currentTool', this.currentTool)
      const { currentTool } = this;
      // 文字备注位置参数
      let textboxLeft1, textboxTop1, textboxLeft2, textboxTop2, textWidth, textHeight = 0;

      // 鼠标移动的长度 绝对值
      const w = Math.abs(this.mouseTo.x - this.mouseFrom.x);
      // 鼠标移动的宽度 绝对值
      const h = Math.abs(this.mouseTo.y - this.mouseFrom.y);
      console.log('this.lineWidthUnit', this.lineWidthUnit)

      // path 路径
      const xf = this.mouseFrom.x;
      const xt = this.mouseTo.x;
      const yf = this.mouseFrom.y;
      const yt = this.mouseTo.y;
      const path = `M${xf} ${yf} L${xf} ${yt} L${xt} ${yt} Z`;

      console.log(path)
      if (currentTool === 'juxing') {
        textboxLeft1 = Math.abs((this.mouseTo.x - this.mouseFrom.x) / 2 + this.mouseFrom.x) - 20;
        textboxTop1 = this.mouseFrom.y - 20;
        textboxLeft2 = this.mouseTo.x + 5;
        textboxTop2 = Math.abs((this.mouseTo.y - this.mouseFrom.y) / 2 + this.mouseFrom.y) - 10;
        textWidth = `${(this.lineWidthUnit * w).toFixed(2)}cm`;
        textHeight = `${(this.lineWidthUnit * h).toFixed(2)}cm`;
      }

      if (currentTool === 'ellipse') {
        textboxLeft1 = this.mouseFrom.x;
        textboxTop1 = this.mouseFrom.y - h - 20;
        textboxLeft2 = this.mouseFrom.x + w + 5;
        textboxTop2 = this.mouseFrom.y - 10;
        // 椭圆周长
        const l = (2 * Math.PI * h + 4 * (w - h)) / 2
        textWidth = `${(this.lineWidthUnit * l).toFixed(2)}cm`;
        textHeight = `${(this.lineWidthUnit * l).toFixed(2)}cm`;
      }

      if (currentTool === 'circle') {
        textboxLeft1 = this.mouseFrom.x + w / 2 + 10;
        textboxTop1 = this.mouseFrom.y - 20;
        textboxLeft2 = this.mouseFrom.x + w + 30;
        textboxTop2 = this.mouseFrom.y + h;
        // 椭圆周长
        const l = Math.PI * w / 2
        textWidth = `${(this.lineWidthUnit * l).toFixed(2)}cm`;
        textHeight = `${(this.lineWidthUnit * l).toFixed(2)}cm`;
      }

      this.textboxObjWidth = new fabric.Textbox(textWidth, {
        left: textboxLeft1,
        top: textboxTop1,
        fontSize: 18,
        fill: this.drawColor,
        hasControls: true,
      });
      this.fabricObj.add(this.textboxObjWidth);

      this.textboxObjHeight = new fabric.Textbox(textHeight, {
        left: textboxLeft2,
        top: textboxTop2,
        fontSize: 18,
        fill: this.drawColor,
        hasControls: true,
      });
      this.fabricObj.add(this.textboxObjHeight);

      let objs = this.fabricObj.getObjects();
      var group = new fabric.Group([this.textboxObjWidth, this.textboxObjHeight, objs[objs.length - 3]], {})

      this.fabricObj.add(group);

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
            fill: "rgba(255, 255, 255, 1)",
          });
          break;
        case "circle": //正圆
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
      console.log('lineWidthUnit', this.lineWidthUnit);

      let objs = this.fabricObj.getObjects();
      console.log('objs', objs)
      objs.map((item, index) => {
        if (item.type === 'group') {
          let height = objs[index - 3].height;
          let width = objs[index - 3].width;
          objs[index - 1].set({
            text: `${(this.lineWidthUnit * height).toFixed(2)}cm`
          })
          objs[index - 2].set({
            text: `${(this.lineWidthUnit * width).toFixed(2)}cm`
          })
        }
      })
      this.fabricObj.remove(objs[objs.length - 1]);
      // this.fabricObj.sendBackwards(objs[objs.length - 2]);
      this.fabricObj.renderAll();
    },
    /* 渲染图片 */
    8 () {
      // 获取图片的原始尺寸
      let img = document.getElementById('test-img');
      let img_url = require('../assets/img/cup.png');
      let image = new Image();
      image.src = img_url;
      console.log('width:' + image.width + ',height:' + image.height);


      // this.fabricObj.setBackgroundImage(require('../assets/img/cup.png'), this.fabricObj.renderAll.bind(this.fabricObj), {
      //   opacity: 1,
      //   width: image.width,
      //   height: image.height,
      //   repeat: 'no-repeat',
      //   // Needed to position backgroundImage at 0/0
      //   originX: 'left',
      //   originY: 'top',
      // });

      let imgInstance = new fabric.Image(img, {
        left: 0, // 图片相对画布的左侧距离
        top: 0, // 图片相对画布的顶部距离
        width: image.width,
        height: image.height,
        repeat: 'no-repeat',
        // angle: 30, // 图片旋转角度
        // opacity: 0.85, // 图片透明度
        // 这里可以通过scaleX和scaleY来设置图片绘制后的大小，这里为原来大小的一半
        scaleX: 0.5,
        scaleY: 0.5
      });
      this.fabricObj.add(imgInstance);
    },
    /* 下移 */
    handleDown () {
      var level = this.fabricObj.getActiveObject();// 获取当前被选中的元素
      this.fabricObj.sendBackwards(level);
    },
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