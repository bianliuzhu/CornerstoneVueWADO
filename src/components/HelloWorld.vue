<template>
  <div>
    <div class="image-canvas-wrapper"  oncontextmenu="return false" unselectable='on' onselectstart='return false;'
      onmousedown='return false;'>
      <!-- DICOM CANVAS -->
        <span id="loadProgress">Diocm加载: </span>
      <div ref="canvas" class="image-canvas" oncontextmenu="return false"></div>
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
  name: "HelloWorld",
  data() {
    return {
      baseUrl: "",
      exampleStudyImageIds: "",
      isInitLoad: true,
      isShow: true
    };
  },
  methods: {
    show() {
      const _this = this;
      if (this.isShow === true) {
        this.isShow = false;
        //this.$http
        //  .get("https://www.baidu.com")
        //  .then(function(res) {
        //    console.log("res:", res);

        //    let Image = res.body.value;
        //    console.log("res.body.value:", res.body.value);

        //    _this.baseUrl = res.body.value.filmain;
        //    console.log("res.body.value.filmain:", res.body.value.filmain);

        //    _this.exampleStudyImageIds = res.body.value.testDate.testDate1;
        //    console.log(
        //      "res.body.value.testDate.testDate1:",
        //      res.body.value.filmain
        //    );
            
            // 找到要渲染的元素
            let canvas = this.$refs.canvas;
            console.log(canvas);
            // 在 DOM 中将 canvas 元素注册到 cornerstone
            cornerstone.enable(canvas);
            // 拼接 url : cornerstoneWADOImageLoader 需要 wadouri 路径头
        //    const imageUrl = _this.baseUrl + _this.exampleStudyImageIds[0];
            //let imageId = "wadouri:" + imageUrl;
            let imageId = "wadouri:static/1.dcm"
            //  Load & Display
            cornerstone.loadAndCacheImage(imageId).then(
              function(image) {
                console.log(image);
                // 设置元素视口
                var viewport = cornerstone.getDefaultViewportForImage(
                  canvas,
                  image
                );
                // 显示图像
                cornerstone.displayImage(canvas, image, viewport);
                // 激活工具
                _this.initCanvasTools();
              },
              function(err) {
                alert(err);
              }
            );

            // Dicom 加载 进度
            cornerstone.events.addEventListener(
              "cornerstoneimageloadprogress",
              function(event) {
                const eventData = event.detail;
                const loadProgress = document.getElementById("loadProgress");
                loadProgress.textContent = `Dicom加载: ${
                  eventData.percentComplete
                }%`;
              }
            );
      }
          //});
      //} else {
      //  this.isShow = true;
      //}
    },
    initCanvasTools() {
      let _self = this;
      let canvas = this.$refs.canvas;
      this.isInitLoad = false;

      // 为 canvasStack 找到 imageIds
      //let allImageIds = [];
      //this.exampleStudyImageIds.forEach(function(imageId) {
      //  let imageUrl = "wadouri:" + _self.baseUrl + imageId;
      //  allImageIds.push(imageUrl);
     // });

      // Create canvasStack
      //let canvasStack = {
      //  currentImageIdIndex: 0,
      //  imageIds: allImageIds
     // };

      // Enable Inputs
      cornerstoneTools.mouseInput.enable(canvas);
      cornerstoneTools.mouseWheelInput.enable(canvas);
      cornerstoneTools.touchInput.enable(canvas);

      // Set the stack as tool state
      cornerstoneTools.addStackStateManager(canvas, ["stack"]);
      //cornerstoneTools.addToolState(canvas, "stack", canvasStack);
      cornerstoneTools.stackScrollWheel.activate(canvas); // Mouse wheel
      cornerstoneTools.scrollIndicator.enable(canvas); // Position indicator

      // Mouse
      cornerstoneTools.wwwc.activate(canvas, 1); // left click
      cornerstoneTools.pan.activate(canvas, 2); // middle click
      cornerstoneTools.zoom.activate(canvas, 4); // right click

      // Touch / Gesture
      cornerstoneTools.wwwcTouchDrag.activate(canvas); // - Drag
      cornerstoneTools.zoomTouchPinch.activate(canvas); // - Pinch
      cornerstoneTools.panMultiTouch.activate(canvas); // - Multi (x2)
    },
    /*
       * Window Resize
       *
       */
    listenForWindowResize() {
      this.$nextTick(function() {
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
      return function() {
        var context = this;
        var args = arguments;
        var later = function() {
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
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.image-canvas-wrapper {
  width: 100%;
  height: 80vh;
  margin: 0 auto;
}

.image-canvas {
  width: 100%;
  height: 100%;
}
</style>
