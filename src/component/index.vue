<template>
  <div class="signatureBox">
    <canvas
      id="signatureCanvas"
      :style="!obj['showCanvasMarsk'] ? 'background: #fff' : 'background: transparent'"
    >Canvas画板</canvas>
    <div v-show="obj['showCanvasMarsk']" class="rt canvasMarsk">
      <div class="write_rl">
        <span v-for="(item, index) in idInfo['form']['name'].split('')" :key="index">
          {{
          item
          }}
        </span>
      </div>

      <p class="tips">
        <span v-for="(item, index) in line1.split('')" :key="index">{{ item }}</span>
      </p>
      <p class="tips">
        <span v-for="(item, index) in line2.split('')" :key="index">{{ item }}</span>
      </p>
    </div>

    <div class="md">
      <div class="write_rl">
        <span v-for="(item, index) in tipText.split('')" :key="index">{{ item }}</span>
      </div>
    </div>

    <div class="lt">
      <button @click="save" @touchstart="save">
        <div class="write_rl">
          <span v-for="(item, index) in confirmSignText.split('')" :key="index">{{ item }}</span>
        </div>
      </button>
      <button @click="clear" @touchstart="clear">
        <div class="write_rl">
          <span v-for="(item, index) in reSignText.split('')" :key="index">{{ item }}</span>
        </div>
      </button>
    </div>
  </div>
</template>
<script>
var draw;
let obj = { showCanvasMarsk: true };

var preHandler = function(e) {
  e.preventDefault();
};

class Draw {
  constructor(el) {
    this.el = el;
    this.canvas = document.getElementById(this.el);
    this.cxt = this.canvas.getContext("2d");
    this.stage_info = this.canvas.getBoundingClientRect();
    this.cxt.translate(0.5, 0.5);
    this.cxt.imageSmoothingEnabled = false;
    this.reset = false;
  }

  init(btn) {
    var that = this;

    this.canvas.addEventListener("touchstart", function(event) {
      document.addEventListener("touchstart", preHandler, false);

      if (obj["showCanvasMarsk"]) {
        obj["showCanvasMarsk"] = false;
      }

      that.drawBegin(event);
    });

    this.canvas.addEventListener("touchend", function(event) {
      document.addEventListener("touchend", preHandler, false);

      that.drawEnd();
    });

    this.clear(btn);
  }

  drawBegin(e) {
    window.console.log("fun--drawBegin");
    var that = this;

    window.getSelection()
      ? window.getSelection().removeAllRanges()
      : document.selection.empty();

    this.cxt.lineWidth = 3; // 6
    this.cxt.strokeStyle = "#333"; // fc1b10

    if (this.reset) {
      this.cxt.beginPath();
      this.reset = false;
    }

    this.cxt.moveTo(
      e.changedTouches[0].clientX - this.stage_info.left,
      e.changedTouches[0].clientY - this.stage_info.top
    );

    this.canvas.addEventListener("touchmove", function() {
      that.drawing(event);
    });
  }

  drawing(e) {
    // window.console.log('fun--drawing');

    this.clear();
    this.cxt.lineTo(
      e.changedTouches[0].clientX - this.stage_info.left,
      e.changedTouches[0].clientY - this.stage_info.top
    );

    this.cxt.stroke();
  }

  drawEnd() {
    window.console.log("fun--drawEnd");
    document.removeEventListener("touchstart", preHandler, false);

    document.removeEventListener("touchend", preHandler, false);

    document.removeEventListener("touchmove", preHandler, false);

    //canvas.ontouchmove = canvas.ontouchend = null
  }

  clear(btn) {
    // window.console.log('fun--clear')
    this.cxt.clearRect(0, 0, this.canvas.width, this.canvas.height);
  }

  save() {
    window.console.log("fun--save");
    return this.canvas.toDataURL("image/png");
  }
}

export default {
  name: "TestBb",
  props: {
    onSuccessBack: {
      type: Function,
      default() {
        return () => {
          return true;
        };
      }
    }
  },
  data() {
    return {
      reSignText: "重签",
      confirmSignText: "确认签名",
      tipText: "我已阅读全部开户文件，了解相关风险并自愿开户",
      obj
    };
  },

  mounted() {
    // 重置状态
    this.obj["showCanvasMarsk"] = true;

    // 先初始化 canvas 的宽高
    const self = this;
    const _canvasEle = document.querySelectorAll(
      "canvas[id='signatureCanvas']"
    )[0];
    const _marskEle = document.querySelectorAll(
      "div[class='rt canvasMarsk']"
    )[0];

    setTimeout(function() {
      self.setCanvasSize(_canvasEle, _marskEle);
    }, 10);

    // 因为 new Draw() 方法里需要获取 canvas 位置坐标，所以需要在 canvas 宽高和位置确定后再执行
    setTimeout(function() {
      draw = new Draw("signatureCanvas");

      draw.init();
    }, 500);
  },

  methods: {
    setCanvasSize(canvas, _marskEle) {
      if (window.devicePixelRatio) {
        console.log("setCanvasSize 1");
        canvas.style.height = _marskEle.offsetHeight + "px";
        canvas.style.width = _marskEle.clientWidth + "px";
        canvas.height = _marskEle.offsetHeight * window.devicePixelRatio;
        canvas.width = _marskEle.clientWidth * window.devicePixelRatio;
        canvas
          .getContext("2d")
          .scale(window.devicePixelRatio, window.devicePixelRatio);
      } else {
        console.log("setCanvasSize 2");
        canvas.height = _marskEle.offsetHeight;
        canvas.width = _marskEle.clientWidth;
      }
    },
    clear: function(e) {
      if (!draw.reset) {
        console.log("clear", new Date().getTime());
        draw.clear();
        obj["showCanvasMarsk"] = true;
        draw.reset = true;
      }
    },
    save() {
      console.log("save", new Date().getTime());
      if (obj["showCanvasMarsk"]) return;

      // 导出原始签名图片
      const _oriImg = draw.save();

      // 将原始签名图片向左旋转 90deg
      this.imgRotate(_oriImg);
    },
    handleSuccess(signatureImg) {
      sessionStorage.setItem("img_ElectronicSignature", signatureImg);
      console.log("img_ElectronicSignature", signatureImg);
      this.$emit("onSuccessBack", signatureImg);
    },
    imgRotate(url, _dir = "left") {
      const _this = this;
      var oc = document.createElement("canvas");
      var ctx,
        image,
        xTimes = 0;
      if (oc.getContext("2d")) {
        ctx = oc.getContext("2d");
        image = new Image();
        image.src = url;
        image.onload = function() {
          if (_dir === "left") {
            xTimes++;
          } else {
            xTimes--;
          }
          ctx.clearRect(0, 0, oc.width, oc.height);
          oc.width = image.height;
          oc.height = image.width;

          var center = {
            x: Math.round(oc.width / 2),
            y: Math.round(oc.height / 2)
          };
          ctx.translate(center.x, center.y);
          ctx.rotate((-90 * xTimes * Math.PI) / 180);

          if (xTimes % 2 !== 0) {
            ctx.translate(-center.y, -center.x);
          } else {
            ctx.translate(-center.x, -center.y);
          }

          ctx.drawImage(image, 0, 0);
          _this.handleSuccess(oc.toDataURL("image/png"));
        };
      }
    }
  },
  computed: {
    idInfo() {
      return {
        form: {
          name: "廖益文"
        }
      };
    },
    line1() {
      return `请安下方图示 【橫向】手写个人【正楷签名】签名：${this.idInfo["form"]["name"]}`;
    },
    line2() {
      return "注意：竖向签名或签名潦草均不能通过审核";
    }
  }
};
</script>
<style lang="scss" scoped>
.signatureBox {
  position: relative;
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  overflow: hidden;

  .lt {
    float: right;
    height: 100%;
    display: flex;
    flex-direction: column;

    button {
      flex: auto;
      margin-bottom: 20px;
      width: 50px;
      color: #409eff;
      background: #ecf5ff;
      border: 1px solid #409eff;
      outline: none;

      .write_rl {
        width: 16px;
        text-align: center;
        margin: 0 auto;
        transform: rotate(180deg);
        transform-origin: center center;

        span {
          display: block;
          transform: rotate(90deg);
          transform-origin: center center;
          font-size: 20px;
          font-weight: bold;
        }
      }

      &:first-child {
        color: #fff;
        background-color: #409eff;
        border-color: #409eff;
      }

      &:last-child {
        margin-bottom: 0;
      }
    }
  }

  .md {
    float: left;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    padding-left: 10px;

    .write_rl {
      width: 16px;
      transform: rotate(180deg);
      transform-origin: center center;

      span {
        display: block;
        transform: rotate(90deg);
        transform-origin: center center;
        color: #999;
        font-size: 12px;
        line-height: 1.2;
      }
    }
  }

  .canvasMarsk {
    position: absolute;
    top: 0;
    left: 0;
    width: 260px;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #fff;

    .write_rl {
      width: 50px;
      margin-right: 0;
      color: #999;
      font-size: 50px;
      font-family: DINBold;
      text-align: center;
    }

    .tips {
      width: 20px;
      color: #999;
      font-size: 18px;

      position: absolute;
      left: 20px;

      span {
        line-height: 1.1;
      }

      &:last-child {
        left: 50px;
      }
    }

    .write_rl,
    .tips {
      transform: rotate(180deg);
      transform-origin: center center;

      span {
        display: block;
        transform: rotate(90deg);
        transform-origin: center center;
      }
    }
  }
  #signatureCanvas {
    float: left;
    background: #fff;
    box-shadow: 0px 0px 9px 0px rgba(0, 0, 0, 0.04);
    border-radius: 4px;
    cursor: default;
    position: relative;
    z-index: 2;
  }
}
</style>