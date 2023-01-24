<template lang="pug">
div(v-if="horizontal")
  div(ref="div" v-show="!loading")
.horizontalizer(v-else)
  div 画面を横向きにしてください
</template>


<script setup lang="ts">
import {
  WebGLRenderer,
  Color,
  Scene,
  Camera,
  MeshNormalMaterial,
  Mesh,
  Group,
  BoxGeometry,
  SphereGeometry
} from "three";
// @ts-ignore
import { ArToolkitSource, ArToolkitContext, ArMarkerControls } from "@ar-js-org/ar.js/three.js/build/ar-threex";
const loading = ref(true)
const div = ref<HTMLDivElement>()
const horizontal = window.innerHeight < window.innerWidth;
const height = window.innerHeight + "px";
if (horizontal) {
  const renderer = new WebGLRenderer({
    alpha: true,
  });
  renderer.setClearColor(new Color(), 0);
  renderer.domElement.style.position = "absolute";
  renderer.domElement.style.top = "0px";
  renderer.domElement.style.left = "0px";

  const scene = new Scene();
  scene.visible = false;
  const camera = new Camera();
  scene.add(camera);

  const arToolkitSource = new ArToolkitSource({
    sourceType: "webcam",
    displayWidth: window.innerWidth,
    displayHeight: window.innerHeight
  });


  arToolkitSource.init(() => {
    setTimeout(() => {
      onResize();
      loading.value = false;
    }, 2000);
  });

  const arToolkitContext = new ArToolkitContext({
    cameraParametersUrl: "data/camera_para.dat",
    detectionMode: 'mono',
    patternRatio: 0.5,
    sourceWidth: window.innerWidth,
    sourceHeight: window.innerHeight,
    canvasWidth: window.innerWidth,
    canvasHeight: window.innerHeight,
    // debug: true
  });

  arToolkitContext.init(() => {
    camera.projectionMatrix.copy(arToolkitContext.getProjectionMatrix());
  });

  const markerN = new Group();
  scene.add(markerN);
  const _arMarkerControlsN = new ArMarkerControls(arToolkitContext, markerN, {
    type: "pattern",
    patternUrl: "data/pattern-nakaya.patt",
    changeMatrixMode: "modelViewMatrix",
  });
  const meshN = new Mesh(new BoxGeometry(), new MeshNormalMaterial());
  meshN.position.y = 0.5;
  markerN.add(meshN);

  const markerY = new Group();
  scene.add(markerY);
  const _arMarkerControlsY = new ArMarkerControls(arToolkitContext, markerY, {
    type: "pattern",
    patternUrl: "data/pattern-yamashita.patt",
    changeMatrixMode: "modelViewMatrix",
  });
  const meshY = new Mesh(new SphereGeometry(), new MeshNormalMaterial());
  meshY.position.y = 0.5;
  markerY.add(meshY);

  onMounted(() => {
    div.value?.appendChild?.(renderer.domElement);
  })
  // const width = ref(window.innerWidth)
  // const height = ref(window.innerHeight)
  const onResize = () => {
    // width.value = window.innerWidth;
    // height.value = window.innerHeight;
    // renderer.setSize(window.innerWidth, window.innerHeight);
    // arToolkitSource.copyElementSizeTo(renderer.domElement)
    // arToolkitSource.onResizeElement();
    const videoTag = document.getElementById("arjs-video") as HTMLVideoElement
    // videoTag.style.width = window.innerWidth + "px";
    // videoTag.style.height = window.innerHeight + "px";
    if (videoTag) {
      // https://teratail.com/questions/188846 より
      // 元の動画のサイズ
      var orgW = videoTag.videoWidth;
      var orgH = videoTag.videoHeight;
      var orgR = orgH / orgW;

      // videoタグのサイズ
      var videoW = videoTag.clientWidth;
      var videoH = videoTag.clientHeight;
      var videoR = videoH / videoW;

      // 描画されている部分のサイズ
      var screenW, screenH;
      if (orgR > videoR) {
        screenH = videoTag.clientHeight;
        screenW = screenH / orgR;
      } else {
        screenW = videoTag.clientWidth;
        screenH = screenW * orgR;
      }
      console.log(screenW, screenH);
      renderer.setSize(screenW, screenH);
      renderer.domElement.style.top = (window.innerHeight - screenH) / 2 + "px";
      renderer.domElement.style.left = (window.innerWidth - screenW) / 2 + "px";
      if (arToolkitContext.arController !== null) {
        // arToolkitSource.copyElementSizeTo(arToolkitContext.arController.canvas)
        arToolkitContext.arController.canvas.style.width = screenW;
        arToolkitContext.arController.canvas.style.height = screenH;
      }
    }
  }
  onResize()
  window.addEventListener("resize", onResize)
  let running = true;
  onUnmounted(() => {
    window.removeEventListener("resize", onResize)
    running = false;
  })

  requestAnimationFrame(function animate() {
    running && requestAnimationFrame(animate);
    if (arToolkitSource.ready) {

      arToolkitContext.update(arToolkitSource.domElement);
      scene.visible = camera.visible;
    }
    renderer.render(scene, camera);
  });

  // console.log(ar)
  // console.log(ArToolkitSource, ArToolkitContext, ArMarkerControls)
}
window.addEventListener("resize", () => {
  if (horizontal !== window.innerHeight < window.innerWidth) {
    location.reload()
  }
})

</script>

<style lang="scss">
body {
  margin: 0;
}

#arjs-video {
  overflow: hidden;
}
</style>

<style scoped lang="scss">
.horizontalizer {
  display: flex;
  justify-content: center;
  align-items: center;
  height: v-bind(height);
  font-size: 3rem;
  text-align: center;
}
</style>