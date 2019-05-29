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
      <div @click="show(5)">
        <img src="../assets/1.png">
        <h4>Grow</h4>
      </div>
      <div @click="renderModel">
        <img src="../assets/1.png">
        <h4>3D</h4>
      </div>
      <div @click="getLength">
        <img src="../assets/1.png">
        <h4>测距</h4>
      </div>
      <div @click="probe">
        <img src="../assets/1.png">
        <h4>探针</h4>
      </div>
      <div @click="arrowAnnotation">
        <img src="../assets/1.png">
        <h4>箭头注释</h4>
      </div>
      <div @click="angle">
        <img src="../assets/1.png">
        <h4>角度工具</h4>
      </div>
      <div @click="freeHand">
        <img src="../assets/1.png">
        <h4>鼠标工具</h4>
      </div>
      <div @click="ellipticalRoi">
        <img src="../assets/1.png">
        <h4>椭圆工具</h4>
      </div>
      <div @click="rectangleRoi">
        <img src="../assets/1.png">
        <h4>矩形工具</h4>
      </div>

      <!-- <div class="block slider-bar">
        <span class="demonstration"></span>
        <el-slider v-model="value1" :format-tooltip="formatTooltip"></el-slider>
      </div>-->
    </div>

    <!-- <div
      class="image-canvas-wrapper"
      oncontextmenu="return false"
      unselectable="on"
      onselectstart="return false;"
      onmousedown="return false;"
    >-->
    <!-- <span id="loadProgress">Diocm加载:</span> -->

    <!-- <div ref="canvas" class="image-canvas" oncontextmenu="return false"></div> -->

    <!-- @cornerstoneimagerendered="onImageRendered" -->

    <div ref="canvas1" class="image-canvas" oncontextmenu="return false" @click="showAxes"></div>

    <div ref="canvas2" class="mask-canvas" oncontextmenu="return false" @click="renderCanvas"></div>

    <!-- <div
        id="topleft"
        class="overlay"
        style="position:absolute;top:0px;left:1vw"
      >姓名：{{patientName}}</div>
      <div
        id="topright"
        class="overlay"
        style="position:absolute;top:0px;right:1vw"
      >渲染时间：{{renderTime}}</div>
      <div
        id="bottomright"
        class="overlay"
        style="position:absolute;bottom:0px;right:1vw"
      >缩放：{{zoom}}</div>
      <div
        id="bottomleft"
        class="overlay"
        style="position:absolute;bottom:0px;left:1vw"
    >WW/WC:{{ratio}}</div>-->

    <!-- <div
        ref="canvas2"
        class="image-canvas-right"
        oncontextmenu="return false"
        @cornerstoneimagerendered="onImageRendered"
    ></div>-->

    <!-- </div> -->

    <br>
    <div class="block slide-bar">
      <span class="demonstration"></span>
      <el-slider :min="1" :max="80" v-model="value1" :format-tooltip="formatTooltip"></el-slider>
    </div>
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
  name: "Index",
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
      value1: 56,
      imageData: [],
      imageDataIndex: 0,
      val: '',
      num: '',
      result: [],
      currentGrayLevel: -1,
    };
  },
  watch: {
    val(val) {
      this.num = this.val
      let _this = this
      this.show(this.num)
    }
  },
  methods: {

    formatTooltip(val) {
      this.val = val
      return val
    },

    onImageRendered(e) {
      const eventData = e.detail;
      cornerstone.setToPixelCoordinateSystem(eventData.enabledElement, eventData.canvasContext);

      this.renderTime = eventData.renderTimeInMs.toFixed(4) + " ms";
      this.zoom = eventData.viewport.scale.toFixed(2);
      this.ratio = Math.round(eventData.viewport.voi.windowWidth) + "/" + Math.round(eventData.viewport.voi.windowCenter);

    },

    showAxes(e) {
      this.result = []
      this.currentGrayLevel = -1
      this.showAxesTemp(e)
      console.log(this.result)
      let indexArr = new Array(230399)
      for (let i = 0; i < 230399; i++) {
        if (this.result.indexOf(i) == -1) {
          indexArr[i] = 255
        } else {
          indexArr[i] = 0
        }
      }
      localStorage.setItem('result', indexArr)
      this.showMask(1)
    },

    showAxesTemp(e, index) {

      let canvas
      let canvasX
      let canvasY

      if (index != undefined) {
        canvasX = index % 480
        canvasY = parseInt(index / 480)
      } else {
        canvas = this.$refs.canvas1
        canvasX = e.clientX - canvas.getBoundingClientRect().left
        canvasY = e.clientY - canvas.getBoundingClientRect().top
        index = canvasX + canvasY * 480
      }

      let low = 75
      let high = 25

      // console.log(index)
      //console.log("当前点击的像素索引值：" + index)
      // console.log("该像素点的灰度值：" + this.imageData[index])
      if (index >= 0 && index < 480 * 480) {
        if (this.currentGrayLevel == -1) {
          this.currentGrayLevel = this.imageData[index]
          this.result.push(index)
        } else {
          if (this.imageData[index] >= this.currentGrayLevel - low && this.imageData[index] <= this.currentGrayLevel + high) {
            this.result.push(index)
          }
        }

        //计算左边点
        let tempIndex = index - 1
        if (tempIndex >= 0 && tempIndex < 480 * 480) {
          if (this.result.indexOf(tempIndex) == -1) {
            if (this.imageData[tempIndex] >= this.currentGrayLevel - low && this.imageData[tempIndex] <= this.currentGrayLevel + high) {
              this.result.push(tempIndex)
              // this.showAxesTemp(e, tempIndex)
            }
          }
        }

        //计算右边点
        tempIndex = index + 1
        if (tempIndex >= 0 && tempIndex < 480 * 480) {

          // for (let i = 0; i < this.result.length; i++) {
          //   if (this.result[i] === tempIndex) {
          //     if (this.imageData[tempIndex] >= this.currentGrayLevel - low && this.imageData[tempIndex] <= this.currentGrayLevel + high) {
          //       this.result.push(tempIndex)
          //       this.showAxesTemp(e, tempIndex)
          //     }
          //   }
          // }

          if (this.result.indexOf(tempIndex) == -1) {
            if (this.imageData[tempIndex] >= this.currentGrayLevel - low && this.imageData[tempIndex] <= this.currentGrayLevel + high) {
              this.result.push(tempIndex)
              this.showAxesTemp(e, tempIndex)
            }
          }

        }

        //计算上边点
        tempIndex = canvasX + (canvasY - 1) * 480
        if (tempIndex >= 0 && tempIndex < 480 * 480) {

          // for (let i = 0; i < this.result.length; i++) {
          //   if (this.result[i] === tempIndex) {
          //     if (this.imageData[tempIndex] >= this.currentGrayLevel - low && this.imageData[tempIndex] <= this.currentGrayLevel + high) {
          //       this.result.push(tempIndex)
          //       this.showAxesTemp(e, tempIndex)
          //     }
          //   }
          // }

          if (this.result.indexOf(tempIndex) == -1) {
            if (this.imageData[tempIndex] >= this.currentGrayLevel - low && this.imageData[tempIndex] <= this.currentGrayLevel + high) {
              this.result.push(tempIndex)
              // this.showAxesTemp(e, tempIndex)
            }
          }

        }

        //计算下边点
        tempIndex = canvasX + (canvasY + 1) * 480
        if (tempIndex >= 0 && tempIndex < 480 * 480) {
          if (this.result.indexOf(tempIndex) == -1) {
            if (this.imageData[tempIndex] >= this.currentGrayLevel - low && this.imageData[tempIndex] <= this.currentGrayLevel + high) {
              this.result.push(tempIndex)
              this.showAxesTemp(e, tempIndex)
            }
          }
        }
        //this.showAxesTemp(e, tempIndex)
      }


      // console.log("top灰度值：" + _this.imageData[topIndex])
      // console.log("bottom灰度值：" + _this.imageData[bottomIndex])
      // console.log("left灰度值：" + _this.imageData[leftIndex])
      // console.log("right灰度值：" + _this.imageData[rightIndex])



    },

    renderCanvas(e) {
      let canvas = this.$refs.canvas2
      var ctx = canvas.getContext("2d");
      console.log(ctx)
      var imgData = ctx.createImageData(100, 100);
      for (var i = 0; i < imgData.data.length; i += 4) {
        imgData.data[i + 0] = 255;
        imgData.data[i + 1] = 0;
        imgData.data[i + 2] = 0;
        imgData.data[i + 3] = 255;
      }
      ctx.putImageData(imgData, 10, 10);

      // let context = canvas.getContext('2d')
      let canvasX = e.clientX - canvas.getBoundingClientRect().left
      let canvasY = e.clientY - canvas.getBoundingClientRect().top
      console.log('X:' + (e.clientX - canvas.getBoundingClientRect().left) + ',Y:' + (e.clientY - canvas.getBoundingClientRect().top))
    },

    show(num) {
      const _this = this;
      var url = 'wadouri:\\static\\dcm\\' + num + '.DCM'
      // console.log(url)
      if (this.isShow === true) {
        // this.isShow = false;
        //  Load & Display
        let canvas = this.$refs.canvas1
        cornerstone.enable(canvas)
        cornerstone.loadAndCacheImage(url).then(
          function (image) {
            // console.log('执行加载DICOM')
            // console.log(image)
            // console.log(image.getPixelData())
            let imageData = image.getPixelData()
            _this.imageData = imageData
            // console.log(_this.imageData)
            // console.log(_this.imageData[_this.imageDataIndex])

            // if (localStorage.getItem('result') != null) {
            //   let indexArr = localStorage.getItem('result').split(",")

            //   if (indexArr.length) {
            //     for (let i in indexArr) {
            //       _this.imageData[i] = indexArr[i]
            //     }
            //   }
            // }

            image.rgba = true

            // 设置元素视口
            var viewport = cornerstone.getDefaultViewportForImage(
              canvas,
              image
            );
            //cornerstoneTools API
            // console.log(cornerstoneTools)
            // 显示图像
            cornerstone.displayImage(canvas, image, viewport);
            cornerstoneTools.mouseInput.enable(canvas);
            cornerstoneTools.mouseWheelInput.enable(canvas);
            // cornerstoneTools.length.activate(canvas,1); 
            // cornerstoneTools.probe.activate(canvas,1); 
            cornerstoneTools.wwwc.activate(canvas, 1); // Left Click
            cornerstoneTools.pan.activate(canvas, 2); // Middle Click
            cornerstoneTools.zoom.activate(canvas, 4); // Right Click
            cornerstoneTools.zoomWheel.activate(canvas); // Mouse Wheel

            // 激活工具
            // _this.initCanvasTools(num);

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

    showMask(num) {
      const _this = this;
      var url = 'wadouri:\\static\\dcm\\' + num + '.DCM'
      if (this.isShow === true) {
        let canvas = this.$refs.canvas2
        cornerstone.enable(canvas)
        cornerstone.loadAndCacheImage(url).then(
          function (image) {
            let imageData = image.getPixelData()
            _this.imageData = imageData
            if (localStorage.getItem('result') != null) {
              let indexArr = localStorage.getItem('result').split(",")
              if (indexArr.length) {
                for (let i in indexArr) {
                  _this.imageData[i] = indexArr[i]
                }
              }
            }
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
          },
          function (err) {
            console.log('加载错误', err)
          }
        );
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

    getLength(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.length.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    probe(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.probe.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    arrowAnnotation(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.arrowAnnotate.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    angle(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.angle.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    freeHand(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.freehand.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    ellipticalRoi(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.ellipticalRoi.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    rectangleRoi(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.rectangleRoi.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    reset(e) {
      let canvas = this.$refs.canvas1
      cornerstone.reset(canvas);
    },

    renderModel() {
      myWindow = window.open('http://www.baidu.com', '百度', 'width=800,height=600')
    },

    initCanvasTools(num) {
      let _self = this;
      let canvas = this.$refs.canvas1;
      this.isInitLoad = false;

      // 为 canvasStack 找到 imageIds

      // let allImageIds = [];
      // this.exampleStudyImageIds.forEach(function (imageId) {
      //   let imageUrl = "wadouri:" + _self.baseUrl + imageId;
      //   allImageIds.push(imageUrl);
      // });

      var url = 'wadouri:\\static\\dcm\\' + num + '.DCM'

      // Create canvasStack
      let canvasStack = {
        currentImageIdIndex: 0,
        imageIds: url
      };

    },

    // Window Resize
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

    // Utility Methods
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

    this.show(56);
    this.showMask(1);

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
  width: 100vw
  height: 50px
  background: black
  position: absolute
  left: 0
  top: 0
  display: flex
  flex-direction: row
  .title-logo
    position: absolute
    top: 0
    left: 0
    img
      width: 50px
      height: 50px
  .title-name
    color: white
    line-height: 40px
    font-size: 18px
    width: 240px
    height: 40px
    margin: 5px 16px 5px 50px
    border-right: 1px solid white
  .patient-list
    color: lightblue
    // background red
    line-height: 42px
    font-size: 18px
    width: 100px
    height: 42px
    margin: 4px 16px 4px 0px
.top-tool-bar
  width: 100vw
  height: 80px
  background: black
  position: absolute
  left: 0
  top: 50px
  padding-top: 10px
  // padding-left 8px
  padding-bottom: 10px
  display: flex
  flex-direction: row
  div
    margin: 0 24px
    cursor: pointer
  h4
    color: white
    font-size: 14px
    line-height: 24px
.image-canvas-wrapper
  width: 100vw
  height: 80vh
  // border 1px solid red
  position: absolute
  top: 150px
  color: orange
  display: flex
  .image-canvas
    // width 100vw
    // height 80vh
    // top 0px
    // left 0px
    border: 4px dashed green !important
.image-canvas-right
  width: 100vw
  height: 80vh
  background: lightgrey
.slide-bar
  width: 200px
  height: 32px
  position: absolute
  left: 120px
  top: 680px
.image-canvas
  width: 480px
  height: 480px
  position: absolute
  top: 150px
  border: 4px dashed green
.mask-canvas
  width: 480px
  height: 480px
  position: absolute
  top: 150px
  left: 500px
  border: 4px dashed blue
</style>
