<template>
  <div class="main">
    <div class="top-title">
      <div class="title-logo">
        <img src="../assets/logo.png">
      </div>
      <div class="title-name">Dicom Viewer Pro By Vf2e</div>
      <div class="patient-list">Patient List</div>
    </div>

    <div class="top-tool-bar">
      <div @click="hflip">
        <img src="../assets/1.png">
        <h4>左翻转</h4>
      </div>
      <div @click="vflip">
        <img src="../assets/1.png">
        <h4>右翻转</h4>
      </div>
      <div @click="lRotate">
        <img src="../assets/1.png">
        <h4>左旋转</h4>
      </div>
      <div @click="rRotate">
        <img src="../assets/1.png">
        <h4>右旋转</h4>
      </div>
      <div @click="reset">
        <img src="../assets/1.png">
        <h4>重置</h4>
      </div>
      <div>
        <img src="../assets/1.png">
        <h4>Grow</h4>
      </div>
      <div @click="renderModel">
        <img src="../assets/1.png">
        <h4>3D</h4>
      </div>
      <div>
        <img src="../assets/1.png">
        <h4>操作</h4>
      </div>
      <div>
        <img src="../assets/1.png">
        <h4>操作</h4>
      </div>
      <div>
        <img src="../assets/1.png">
        <h4>操作</h4>
      </div>
      <div>
        <img src="../assets/1.png">
        <h4>操作</h4>
      </div>
      <div>
        <img src="../assets/1.png">
        <h4>操作</h4>
      </div>
      <div>
        <img src="../assets/1.png">
        <h4>操作</h4>
      </div>
      <div>
        <img src="../assets/1.png">
        <h4>操作</h4>
      </div>
      <div>
        <img src="../assets/1.png">
        <h4>操作</h4>
      </div>
      <div>
        <img src="../assets/1.png">
        <h4>操作</h4>
      </div>
      <div>
        <img src="../assets/1.png">
        <h4>操作</h4>
      </div>
    </div>

    <div
      class="image-canvas-wrapper"
      oncontextmenu="return false"
      unselectable="on"
      onselectstart="return false;"
      onmousedown="return false;"
    >
      <!-- <span id="loadProgress">Diocm加载:</span> -->

      <!-- <div ref="canvas" class="image-canvas" oncontextmenu="return false"></div> -->

      <div
        ref="canvas1"
        class="image-canvas"
        oncontextmenu="return false"
        @cornerstoneimagerendered="onImageRendered"
      >
        <div
          id="topleft"
          class="overlay"
          style="position:absolute;top:0px;left:1vw"
        >姓名：{{patientName}}</div>
        <div
          id="topright"
          class="overlay"
          style="position:absolute;top:0px;right:51vw"
        >渲染时间：{{renderTime}}</div>
        <div
          id="bottomright"
          class="overlay"
          style="position:absolute;bottom:0px;right:51vw"
        >缩放：{{zoom}}</div>
        <div
          id="bottomleft"
          class="overlay"
          style="position:absolute;bottom:0px;left:1vw"
        >WW/WC:{{ratio}}</div>
      </div>
      <div
        ref="canvas2"
        class="image-canvas-right"
        oncontextmenu="return false"
        @cornerstoneimagerendered="onImageRendered"
      ></div>
    </div>

    <br>
    <!-- <el-button id="hFlip" @click="hflip" class="btn btn-default">HFlip</el-button>
    <el-button id="vFlip" @click="vflip" class="btn btn-default">VFlip</el-button>
    <el-button id="lRotate" @click="lRotate" class="btn btn-default">Rotate Left</el-button>
    <el-button id="rRotate" @click="rRotate" class="btn btn-default">Rotate Right</el-button>
    <el-button id="reset" @click="reset" class="btn btn-default">Reset</el-button>
    <el-button id="renderModel" @click="renderModel" class="btn btn-default">3D</el-button>-->

    <br>
    <!-- <div class="block slide-bar">
      <span class="demonstration"></span>
      <el-slider v-model="value1" :format-tooltip="formatTooltip"></el-slider>
    </div>-->
  </div>
</template>

<script>
//引入 cornerstone,dicomParser,cornerstoneWADOImageLoader
import * as cornerstone from "cornerstone-core";
import * as dicomParser from "dicom-parser";

// 不建议 npm 安装 cornerstoneWADOImageLoader 如果你做了 会很头疼
import * as cornerstoneWADOImageLoader from "../../static/dist/cornerstoneWADOImageLoader.js";

// Cornerstone 工具外部依赖
import Hammer from "hammerjs";
import * as cornerstoneMath from "cornerstone-math";
import * as cornerstoneTools from "cornerstone-tools";

// Specify external dependencies
cornerstoneTools.external.Hammer = Hammer;
cornerstoneTools.external.cornerstone = cornerstone;
cornerstoneTools.external.cornerstoneMath = cornerstoneMath;
cornerstoneWADOImageLoader.external.cornerstoneMath = cornerstoneMath;

//指定要注册加载程序的基石实例
cornerstoneWADOImageLoader.external.cornerstone = cornerstone;

cornerstone.registerImageLoader("http", cornerstoneWADOImageLoader.loadImage);
cornerstone.registerImageLoader("https", cornerstoneWADOImageLoader.loadImage);

//配置 webWorker (必须配置)
//注意这里的路径问题  如果路径不对 cornerstoneWADOImageLoaderWebWorker 会报错 index.html Uncaught SyntaxError: Unexpected token <
var config = {
  webWorkerPath: "/static/dist/cornerstoneWADOImageLoaderWebWorker.js",
  taskConfiguration: {
    decodeTask: {
      codecsPath: "/static/dist/cornerstoneWADOImageLoaderCodecs.js"
    }
  }
};
cornerstoneWADOImageLoader.webWorkerManager.initialize(config);

export default {
  name: "HelloWorld",
  data() {
    return {
      baseUrl: "",
      exampleStudyImageIds: "",
      isInitLoad: true,
      isShow: true,
      patientName: 'wangchang',
      renderTime: '',
      zoom: '',
      ratio: '',
      value1: 0,
    };
  },
  methods: {

    formatTooltip(val) {
      return val / 100;
    },

    onImageRendered(e) {
      const eventData = e.detail;
      cornerstone.setToPixelCoordinateSystem(eventData.enabledElement, eventData.canvasContext);

      const context = eventData.canvasContext;
      context.beginPath();
      context.strokeStyle = 'yellow';
      context.lineWidth = .5;
      context.rect(128, 90, 150, 60);
      context.stroke();
      context.fillStyle = "orange";
      context.font = "8px";
      context.fillText("canvas标注测试", 128, 85);

      context.beginPath();
      context.arc(256, 256, 80, 0, 2 * Math.PI, false);
      context.fillStyle = 'lightblue';
      context.lineWidth = 1;
      context.font = "8px";
      context.fillText("canvas标注测试", 136, 185);
      context.strokeStyle = 'red';
      context.stroke();

      this.renderTime = eventData.renderTimeInMs + " ms";
      this.zoom = eventData.viewport.scale.toFixed(2);
      this.ratio = Math.round(eventData.viewport.voi.windowWidth) + "/" + Math.round(eventData.viewport.voi.windowCenter);

    },
    show() {
      const _this = this;
      var url = `wadouri:\\static\\dcm\\1.DCM`
      if (this.isShow === true) {
        this.isShow = false;
        //  Load & Display
        let canvas = this.$refs.canvas1
        cornerstone.enable(canvas)
        cornerstone.loadAndCacheImage(url).then(
          function (image) {
            console.log(image);

            // 设置元素视口
            var viewport = cornerstone.getDefaultViewportForImage(
              canvas,
              image
            );
            // 显示图像
            cornerstone.displayImage(canvas, image, viewport);
            cornerstoneTools.mouseInput.enable(canvas);
            cornerstoneTools.mouseWheelInput.enable(canvas);
            cornerstoneTools.wwwc.activate(canvas, 1); // Left Click
            cornerstoneTools.pan.activate(canvas, 2); // Middle Click
            cornerstoneTools.zoom.activate(canvas, 4); // Right Click
            cornerstoneTools.zoomWheel.activate(canvas); // Mouse Wheel
            // 激活工具
            _this.initCanvasTools();

            // const ProbeTool = cornerstoneTools.ProbeTool;
            // cornerstoneTools.addTool(ProbeTool)
            // cornerstoneTools.setToolActive('Probe', { mouseButtonMask: 1 })

          },
          function (err) {
            console.log('加载错误', err)
          }
        );
        // Dicom 加载 进度
        // cornerstone.events.addEventListener(
        //   "cornerstoneimageloadprogress",
        //   function (event) {
        //     const eventData = event.detail;
        //     const loadProgress = document.getElementById("loadProgress");
        //     loadProgress.textContent = `Dicom文件加载: ${
        //       eventData.percentComplete
        //       }%`;
        //   }
        // )
      }
    },

    hflip(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      viewport.hflip = !viewport.hflip;
      cornerstone.setViewport(canvas, viewport);
    },

    vflip(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      viewport.vflip = !viewport.vflip;
      cornerstone.setViewport(canvas, viewport);
    },

    lRotate(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      viewport.rotation -= 90;
      cornerstone.setViewport(canvas, viewport);
    },

    rRotate(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      viewport.rotation += 90;
      cornerstone.setViewport(canvas, viewport);
    },

    reset(e) {
      let canvas = this.$refs.canvas1
      cornerstone.reset(canvas);
    },

    renderModel() {
      myWindow = window.open('http://www.baidu.com', '百度', 'width=800,height=600')
    },

    initCanvasTools() {
      let _self = this;
      let canvas = this.$refs.canvas;
      this.isInitLoad = false;

      // 为 canvasStack 找到 imageIds

      // let allImageIds = [];
      // this.exampleStudyImageIds.forEach(function (imageId) {
      //   let imageUrl = "wadouri:" + _self.baseUrl + imageId;
      //   allImageIds.push(imageUrl);
      // });

      var url = `wadouri:\\static\\dcm\\1.DCM`

      // Create canvasStack
      let canvasStack = {
        currentImageIdIndex: 0,
        imageIds: url
      };

      // // Enable Inputs
      // cornerstoneTools.mouseInput.enable(canvas);
      // cornerstoneTools.mouseWheelInput.enable(canvas);
      // cornerstoneTools.touchInput.enable(canvas);

      // // Set the stack as tool state
      // cornerstoneTools.addStackStateManager(canvas, ["stack"]);
      // cornerstoneTools.addToolState(canvas, "stack", canvasStack);
      // cornerstoneTools.stackScrollWheel.activate(canvas); // Mouse wheel
      // cornerstoneTools.scrollIndicator.enable(canvas); // Position indicator

      // // Mouse
      // cornerstoneTools.wwwc.activate(canvas, 1); // left click
      // cornerstoneTools.pan.activate(canvas, 2); // middle click
      // cornerstoneTools.zoom.activate(canvas, 4); // right click

      // // Touch / Gesture
      // cornerstoneTools.wwwcTouchDrag.activate(canvas); // - Drag
      // cornerstoneTools.zoomTouchPinch.activate(canvas); // - Pinch
      // cornerstoneTools.panMultiTouch.activate(canvas); // - Multi (x2)

    },
    /*
       * Window Resize
       *
       */
    listenForWindowResize() {
      this.$nextTick(function () {
        window.addEventListener(
          "resize",
          this.debounce(this.onWindowResize, 100)
        );
      });
    },
    onWindowResize() {
      cornerstone.resize(this.$refs.canvas, true);
    },
    /*
       * Utility Methods
       *
       */
    debounce(func, wait, immediate) {
      var timeout;
      return function () {
        var context = this;
        var args = arguments;
        var later = function () {
          timeout = null;
          if (!immediate) func.apply(context, args);
        };
        var callNow = immediate && !timeout;
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
        if (callNow) func.apply(context, args);
      };
    }
  },
  mounted() {
    this.show();

    // this.$refs.canvas.addEventListener('cornerstoneimagerendered', onImageRendered);

    // cornerstone.events.addEventListener('cornerstoneimageloaded', function(e) {
    //     const eventData = e.detail;
    //     document.getElementById('imageLoaded').textContent = `Last ImageId Loaded: ${eventData.image.imageId}`;
    // });

  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" scoped>
.top-title
  width 100vw
  height 50px
  background black
  position absolute
  left 0
  top 0
  display flex
  flex-direction row
  .title-logo
    position absolute
    top 0
    left 0
    img
      width 50px
      height 50px
  .title-name
    color white
    line-height 40px
    font-size 18px
    width 200px
    height 40px
    margin 5px 16px 5px 50px
    border-right 1px solid white
  .patient-list
    color lightblue
    // background red
    line-height 42px
    font-size 18px
    width 100px
    height 42px
    margin 4px 16px 4px 0px
.top-tool-bar
  width 100vw
  height 80px
  background black
  position absolute
  left 0
  top 50px
  padding-top 10px
  // padding-left 8px
  padding-bottom 10px
  display flex
  flex-direction row
  div
    margin 0 24px
    cursor pointer
  h4
    color white
    font-size 14px
    line-height 24px
.image-canvas-wrapper
  width 100vw
  height 80vh
  // border 1px solid red
  position absolute
  top 130px
  color orange
  display flex
  .image-canvas
    width 50vw
    height 80vh
    top 0px
    left 0px
    // border 4px dashed green
.image-canvas-right
  width 50vw
  height 80vh
  background lightgrey
</style>
