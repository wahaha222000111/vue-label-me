<template>
  <div class="wraper" ref="wraper">
    <div class="operate">
      <ul>
        <li><label>颜色：</label>
          <el-color-picker v-model="drawColor"></el-color-picker>
        </li>
      </ul>
      <ul v-show="currentClass==='lineSegment'">
        <li>
          <label>样式：</label>
          <el-select v-model="currentTool" placeholder="请选择">
            <el-option v-for="item in lineSegmentList" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
        </li>
      </ul>
      <ul v-show="currentClass==='polygon'">
        <li>
          <label>样式：</label>
          <el-select v-model="currentTool" placeholder="请选择">
            <el-option v-for="item in polygonList" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
        </li>
      </ul>
      <ul v-show="currentTool==='pencil'">
        <li>
          <label>粗细：</label>
          <el-input style="width: 100px" type="number" v-model="pencilWeight"></el-input>
        </li>
      </ul>
      <ul v-show="currentTool==='text'">
        <!--    <li><label >颜色：</label><el-color-picker v-model="drawColor"></el-color-picker></li>-->
        <li>
          <label>大小：</label>
          <el-input style="width: 100px" type="number" v-model="fontSize"></el-input>
        </li>
        <li>
          <label>字体：</label>
          <el-select style="width: 100px" v-model="fontFamily" placeholder="请选择">
            <el-option v-for="item in fontFamilyList" :key="item.en" :label="item.ch" :value="item.en">
            </el-option>
          </el-select>
        </li>
        <li>
          <label>粗细：</label>
          <el-select style="width: 100px" v-model="fontWeight" placeholder="请选择">
            <el-option v-for="item in fontWeightList" :key="item" :value="item">
              {{item}}
            </el-option>
          </el-select>
        </li>
        <li>
          <label>样式：</label>
          <el-select style="width: 100px" v-model="fontStyle" placeholder="请选择">
            <el-option v-for="item in [{label:'正常',value:'normal'},{label:'斜体',value:'oblique'}]" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
        </li>
      </ul>
      <el-button class="upload-btn" v-show="showUploadBtn" type="primary" @click.stop="uploadImage">上传图片</el-button>
    </div>

    <div class="canvas-wraper">
      <canvas id="canvas" ref="canvas"></canvas>
      <img id="test-img" src="../assets/img/kotei_9628.png" style="display: none">
    </div>
    <div class="controlPanel">
      <div :class="[initIdx===idx ? 'contro-item active' : 'contro-item']" v-for="(item,idx) in toolsArr" :key="idx" @click="handleTools(item, idx)">
        <!--        <i :class="'iconfont' + item.icon"></i>-->
        <span>{{item.title}}</span>
      </div>
    </div>
    <div class="download">
      <button type="button" :disabled="done" @click="downLoadImage">转换为base64并预览</button>
    </div>

    <el-dialog :visible.sync="visible" title="预览" @before-close="()=>{visible = false}">
      <img width="100%" :src="imageBase64" v-show="imageBase64!=''" alt="">
    </el-dialog>
  </div>
</template>
<script>
import { fabric } from 'fabric'
export default {
  data () {
    return {
      visible: false,
      currentTool: '',
      currentClass: '',
      ableEdit: false,
      done: false,
      fabricObj: null,
      initIdx: 0,
      toolsArr: [
        { name: 'pencil', icon: ' icon-pencil', title: "画笔" },
        { name: 'lineSegment', title: '线段' },
        // {name: 'line', icon: ' icon-line',title:"直线"},
        // {name: 'arrow', icon: ' icon-arrow',title:"箭头"},
        // {name: 'xuxian', icon: ' icon-xuxian',title:"虚线"},
        { name: 'text', icon: ' icon-ziti', title: '文本' },
        { name: 'polygon', title: '多边形' },
        // {name: 'juxing', icon: ' icon-juxing',title:"矩形"},
        // {name: 'cricle', icon: ' icon-yuanxing',title:'圆形'},
        // {name: 'ellipse', icon: ' icon-tuoyuanxing',title:'椭圆'},
        // {name: 'equilateral',icon: ' icon-sanjiaoxing',title:"三角形"},
        { name: 'image', icon: 'icon-image', title: '图片' },
        { name: 'edit', title: '编辑' },
        { name: 'remove', icon: ' icon-remove', title: "删除" },
        { name: 'undo', icon: ' icon-huitui', title: "上一步" },
        { name: 'redo', icon: ' icon-xiangqian', title: "下一步" },
        { name: 'reset', icon: ' icon-reset', title: '清空' },
      ],
      showUploadBtn: false,
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
      fontFamily: window.getComputedStyle(document.body)['fontFamily'],
      fontFamilyList: [],
      fontWeight: 400,
      fontWeightList: [100, 200, 300, 400, 500, 600, 700, 800],
      fontStyle: 'normal',
      fontSize: 16,
      lineSegmentList: [],
      polygonList: [],
      pencilWeight: 1,
    }
  },
  computed: {
    canvasWidth () {
      return window.innerWidth
    },
    pencilPreview () {
      return {
        color: this.drawColor,
        width: this.pencilWeight
      }
    }
  },
  watch: {
    pencilPreview (newVal) {
      this.fabricObj.freeDrawingBrush.color = newVal.color
      this.fabricObj.freeDrawingBrush.width = newVal.width
    }
  },
  created () {
  },
  mounted () {
    this.lineSegmentList = [
      { label: '直线', value: 'line', url: '' },
      { label: '虚线', value: 'xuxian', url: '' },
      { label: '箭头', value: 'arrow', url: '' },
      { label: '点划线', value: 'dotLine', url: '' },
    ]
    this.polygonList = [
      { label: '矩形', value: 'juxing', url: '' },
      { label: '圆形', value: 'cricle', url: '' },
      { label: '椭圆', value: 'ellipse', url: '' },
      { label: '三角形', value: 'equilateral', url: '' },
    ]
    this.fontFamilyList = [{
      ch: '宋体',
      en: '"SimSun"'
    }, {
      ch: '黑体',
      en: '"SimHei"'
    }, {
      ch: '微软雅黑',
      en: '"Microsoft YaHei"'
    }, {
      ch: '微软正黑体',
      en: '"Microsoft JhengHei"'
    }, {
      ch: '楷体',
      en: '"KaiTi"'
    }, {
      ch: '新宋体',
      en: '"NSimSun"'
    }, {
      ch: '仿宋',
      en: '"FangSong"'
    }]
    //初始化canvas
    this.initCanvas()
  },
  methods: {
    /**
     * @desc 编辑事件
     * */
    edit () {
      // let dataJson = this.fabricHistoryJson.map(item=>JSON.parse(item))
      // console.log(dataJson)
      this.ableEdit = true;
      this.doDrawing = false;
      // this.currentTool = ''
      this.fabricObj.isDrawingMode = false;
      this.fabricObj.selectable = true;
      this.fabricObj.selection = true;
      this.fabricObj.skipTargetFind = false;
    },
    initCanvas () {
      this.fabricObj = new fabric.Canvas('canvas', {
        // isDrawingMode: false,
        // selectable: true,
        // selection: true,
        devicePixelRatio: true, //Retina 高清屏 屏幕支持
      });
      // let img = document.getElementById('test-img');
      this.fabricObj.freeDrawingBrush.color = this.drawColor
      // this.fabricObj.freeDrawingBrush.width = 1
      this.fabricObj.setWidth(this.canvasWidth)
      this.fabricObj.setHeight(500);
      this.fabricObj.add(
        // new fabric.Rect({ top: 100, left: 100, width: 50, height: 50, fill: '#f55' }),
        // new fabric.Circle({ top: 140, left: 230, radius: 75, fill: 'green' }),
        // new fabric.Triangle({ top: 300, left: 210, width: 100, height: 100, fill: 'blue' }),
        // new fabric.Image(img,{top:0,left:0,width:500,height:500}),
      );
      /*  let imgInstance = new fabric.Image(img, {
            left: 0,
            top: 0,
           /!* width:1000,
            height:500,*!/
            // angle: 30,
            opacity: 1
        });
            this.fabricObj.add(imgInstance);*/
      //绑定画板事件
      this.fabricObjAddEvent()
    },
    //时间监听
    fabricObjAddEvent () {
      this.fabricObj.on({
        'mouse:down': (o) => {
          // if (this.ableEdit){
          //   return
          // }
          // this.doDrawing = true;
          // // e.preventDefault();
          // //鼠标按下
          // console.log('mouse down')
          // this.mouseFrom.x = o.pointer.x;
          // this.mouseFrom.y = o.pointer.y;

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
          if (this.currentTool !== 'text') {
            this.updateModifications(true);
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
        //对象移动时间
        'object:moving': (e) => {
          e.target.opacity = 0.5;
        },
        //增加对象
        'object:added': (e) => {
          // debugger
        },
        'object:modified': (e) => {
          console.log('object:modified')
          e.target.opacity = 1;
          let object = e.target;
          this.updateModifications(true)
        },
        'selection:created': (e) => {
          if (this.currentTool === 'remove') {
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
      this.currentClass = tools.name;
      this.currentTool = tools.name;
      this.fabricObj.isDrawingMode = false;
      this.showUploadBtn = false
      this.resetObj()
      switch (tools.name) {
        case 'pencil':
          this.currentTool = 'pencil';
          this.fabricObj.isDrawingMode = true;
          break;
        case 'lineSegment':
          this.currentTool = 'line';
          break;
        case 'polygon':
          this.currentTool = 'juxing';
          break;
        case 'image':
          this.showUploadBtn = true
          break;
        case 'edit':
          this.edit()
          break;
        case 'remove':
          this.fabricObj.selection = true
          this.fabricObj.skipTargetFind = false
          this.fabricObj.selectable = true
          break;
        case 'reset':
          this.fabricObj.clear();
          this.updateModifications(true)
          break;
        case 'redo':
          this.$nextTick(() => {
            this.redo();
          })
          break;
        case 'undo':
          this.$nextTick(() => {
            this.undo();
          })
          break;
        default:
          break;
      }
    },
    //绘制文字对象
    drawText () {
      // 创建字体对象
      this.textboxObj = new fabric.Textbox(" ", {
        left: this.mouseFrom.x,
        top: this.mouseFrom.y,
        width: 100,
        fontSize: this.fontSize, // 字号
        fontWeight: this.fontWeight,
        fontFamily: this.fontFamily,
        fontStyle: this.fontStyle,
        // textAlign:'center',
        // originX:'left',
        fill: this.drawColor, // 填充色
        hasControls: true,
        borderColor: 'orange',
        editingBorderColor: 'blue' // 点击
      });
      this.fabricObj.add(this.textboxObj);
      this.textboxObj.enterEditing();
      this.textboxObj.hiddenTextarea.focus();
      // this.updateModifications(true)
    },

    drawing () {
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
          })
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
        case 'dotLine': //doshed line
          fabricObject = new fabric.Line([this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y], {
            strokeDashArray: [10, 3, 3, 3],
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
            fill: "rgba(255, 255, 255, 0)"
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
      this.visible = true
    },
    async uploadImage () {
      let imageMessage = null
      const [fileHandle] = await window.showOpenFilePicker({
        types: [
          {
            description: 'Images',
            accept: {
              'image/*': ['.png', '.gif', '.jpeg', '.jpg']
            }
          },
        ],
        excludeAcceptAllOption: true,
        multiple: false
      })
      imageMessage = await fileHandle.getFile()

      // let imageELement = document.createElement('img')
      // imageELement.src= await this.getBase64(imageMessage)
      let image = new Image()
      image.src = await this.getBase64(imageMessage);
      image.onload = () => {
        let scaleX = (image.width / this.$refs.canvas.clientWidth).toFixed(2)
        let scaleY = (image.height / this.$refs.canvas.clientHeight).toFixed(2)
        console.log(scaleX, scaleY)
        let scale = (Math.abs(1 - (scaleX > scaleY ? scaleY : scaleX))).toFixed(2)

        console.log(scale)

        const imageObj = new fabric.Image(image, {
          left: 10, // 图片相对画布的左侧距离
          top: 60, // 图片相对画布的顶部距离
          angle: 0, // 图片旋转角度
          opacity: 0.85, // 图片透明度
          // 这里可以通过scaleX和scaleY来设置图片绘制后的大小，这里为原来大小的一半
          scaleX: scale,
          scaleY: scale,
          hasControls: true,
        });
        imageObj.set({
          // hasControls:true,
          // transparentCorners: false,
          // cornerColor: 'blue',
          // cornerStrokeColor: 'red',
          // borderColor: 'red',
          // cornerSize: 12,
          // padding: 10,
          // cornerStyle: 'circle',
          // borderDashArray: [3, 3]
        });
        // 添加对象后, 如下图
        // this.fabricObj.isDrawingMode =  true
        this.fabricObj.add(imageObj)
      }
    },
    // 获取文件的base64信息
    getBase64 (file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader()
        //开始读文件
        //readAsDataURL: dataurl它的本质就是图片的二进制数据， 进行base64加密后形成的一个字符串，
        reader.readAsDataURL(file)
        // 成功和失败返回对应的信息，reader.result一个base64，可以直接使用
        reader.onload = () => resolve(reader.result)
        // 失败返回失败的信息
        reader.onerror = error => reject(error)
      })
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
  height: 40px;
  border-right: 1px solid #dad7d9;
  text-align: center;
  cursor: pointer;
  background: #fefefe;
}
.wraper .controlPanel .contro-item span {
  font-size: 16px;
  line-height: 40px;
}
.wraper .controlPanel .contro-item.active {
  background: #e34f51;
}
.wraper .controlPanel .contro-item.active span {
  color: #fff;
  border-radius: 3px;
  font-size: 20px;
}
.download img {
  width: 100%;
}
.operate {
  display: flex;
  height: 60px;
  background-color: paleturquoise;
  border-radius: 10px;
}
.operate > ul {
  padding: 0 10px;
  display: flex;
  align-items: center;
}
.operate > ul > li {
  margin-right: 10px;
  height: 60px;
  display: flex;
  align-items: center;
}
/*.upload-btn{*/
/*  position: fixed;*/
/*  left: 10px;*/
/*  top: 60px;*/
/*  z-index: 1;*/
/*}*/
</style>
