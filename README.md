# ElectronicSignature
基于 Vue 的电子签名组件

## 一、通过 Node 引用

```javascript
npm i vue-electronicsignature
```

在 VUE 的 SPA 中的使用示例：

```html
<template>
  <div id="main">
    <div id="navBar">电子签名</div>
    <div class="signature-box" :style="signatureStyle">
      <ElectronicSignature @onSuccessBack="handleSuccess" />
    </div>
  </div>
</template>
<script>
import ElectronicSignature from "vue-electronicsignature";
export default {
  name: "Signature",
  components: { ElectronicSignature },
  data() {
    return {
      signatureStyle: ""
    };
  },
  mounted() {
    this.$nextTick(() => {
      // 固定 topBar 的位置
      const topBarEle = document.querySelectorAll("div[id='navBar']")[0];
      const _topBarH = topBarEle.clientHeight;

      // 初始化签名组件的样式
      const _windowH = window.innerHeight;
      const _windowW = window.innerWidth;
      const _containerH = _windowH - _topBarH;
      this.signatureStyle = "height:" + _containerH + "px;" + "width:" + _windowW + "px;" + "top:" + _topBarH + "px";
    });
  },
  methods: {
    handleSuccess(img) {
      alert(img);
    }
  }
};
</script>
<style lang="scss" scoped>
#main {
  min-height: 100vh;
  width: 100%;
  background: #f5f5f5;
  overflow: hidden;
  #navBar {
    width: 100%;
    height: 50px;
    background-color: #fff;
    text-align: center;
    font-size: 18px;
    line-height: 48px;
    color: #444;
    font-weight: bold;
    position: fixed;
    top: 0;
    left: 0;
  }
  .signature-box {
    position: fixed;
    top: 100px;
    left: 0;
    width: 100%;
    padding: 15px;
    background-color: #f5f5f5;
  }
}
</style>
```

## 二、示例

[Demo](https://alvinyw.github.io/Blog/mobile-demo/#/electronicSignature)