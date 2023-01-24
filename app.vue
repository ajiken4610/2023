<template lang="pug">
div(ref="div")
</template>


<script setup lang="ts">
import {
  WebGLRenderer,
  Color,
  Scene,
  Camera,
  MeshNormalMaterial,
  Mesh,
  Clock,
  Group,
  BoxGeometry,
} from "three";
// @ts-ignore
import { ArToolkitSource, ArToolkitContext, ArMarkerControls } from "@ar-js-org/ar.js/three.js/build/ar-threex";

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
  }, 2000);
});

const arToolkitContext = new ArToolkitContext({
  cameraParametersUrl: "data/camera_para.dat",
  patternRatio: 0.5,
  sourceWidth: window.innerWidth,
  sourceHeight: window.innerHeight,
  canvasWidth: window.innerWidth,
  canvasHeight: window.innerHeight,
  debug: true
});

arToolkitContext.init(() => {
  camera.projectionMatrix.copy(arToolkitContext.getProjectionMatrix());
});

const marker = new Group();
scene.add(marker);
const arMarkerControls = new ArMarkerControls(arToolkitContext, marker, {
  type: "pattern",
  patternUrl: "data/pattern-nakaya.patt",
  changeMatrixMode: "modelViewMatrix",
});
const mesh = new Mesh(new BoxGeometry(), new MeshNormalMaterial());
mesh.position.y = 0.5;
marker.add(mesh);

const div = ref<HTMLDivElement>()
onMounted(() => {
  div.value?.appendChild?.(renderer.domElement);
})
// const width = ref(window.innerWidth)
// const height = ref(window.innerHeight)
const onResize = () => {
  // width.value = window.innerWidth;
  // height.value = window.innerHeight;
  renderer.setSize(window.innerWidth, window.innerHeight);
  arToolkitSource.onResizeElement();
  if (arToolkitContext.arController !== null) {
    arToolkitContext.arController.canvas.style.width = window.innerWidth;
    arToolkitContext.arController.canvas.style.height = window.innerHeight;
  }
}
onResize()
window.addEventListener("resize", onResize)
onUnmounted(() => {
  window.removeEventListener("resize", onResize)
})

// console.log(ar)
// console.log(ArToolkitSource, ArToolkitContext, ArMarkerControls)

</script>

<style lang="scss">
body {
  margin: 0;
}

#arjs-video {
  overflow: hidden;
}
</style>