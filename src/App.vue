<script setup>
import svgs from '/src/assets/naml.svg?raw'
import img from '/src/assets/b_94c9fefceb8293e508d07575d869622a.jpg'
import svg2 from '/src/assets/cover_98698741_p0-01vS6CJ4.svg?raw'
import img2 from '/src/assets/cover_98698741_p0-FOC34k7N.jpg'
import svg3 from '/src/assets/7658A8862B07D9DF8141A336786A9DE3.svg?raw'
import img3 from '/src/assets/7658A8862B07D9DF8141A336786A9DE3.jpg'
import {nextTick, reactive,ref} from "vue";
const v = reactive([]);
const showImg = ref(false)
const imgWidth = ref(0)
const imgHeight = ref(0)
const imgUrl = ref('')
const data = [
  {
    svg: svgs,
    img:img
  },{
  svg: svg2,
    img: img2
  },{
  svg: svg3,
    img: img3
  }
]
start()
async function start(){
  for (;;){
    for (const item of data) {
      await animation(item.svg,item.img)
      await sleep(1000)
    }
  }
}
function animation(svgStr,iu){
  return new Promise(async resolve =>{
    imgUrl.value = iu
    const [width,height] = await readImgWidthHeight(iu)
    showImg.value = false
    imgWidth.value = width
    imgHeight.value = height
    const g = new DOMParser().parseFromString(svgStr, "image/svg+xml").getElementsByTagName("path")
    v.length = 0
    for (let S = 0; S < g.length; S++){
      let obj = {
        d: g[S].getAttribute("d"),
        index: S,
        color: randomColor()
      }
      v.push(obj)
    }
    nextTick(()=>{
      v.forEach(item=>{
        let path = document.getElementById("path"+item.index);
        item.length = path.getTotalLength();
        item.offset = item.length
      })
      v.sort(()=>{
        return Math.random() - 0.5;
      })
      startAnimation().then(async res=>{
        await sleep(500)
        showImg.value=true
        resolve()
      })
    })
  })
}
function randomColor(){
  return Math.random() <= .5 ? "rgb(19, 234, 244)" : "rgb(225, 26, 157)"
}

function startAnimation(){
  return new Promise(async pResolve=>{
    sleep(400)
    for (let itemIndex in v) {
      let item = v[itemIndex]
      let isLast = ((Number(itemIndex)+1) === v.length)
      new Promise((resolve)=>{
        let inter = setInterval(()=>{
          let subT = item.length/10
          item.offset-=subT
          if (item.offset <= 0){
            // item.offset = item.length
            item.offset = 0
            item.color = 'white'
            clearInterval(inter)
            resolve()
            if (isLast){
              pResolve()
            }
            // let inter2 = setInterval(()=>{
            //   item.offset -= subT
            //   if (item.offset <= 0){
            //     item.offset = 0
            //     clearInterval(inter2)
            //   }
            // },5)
          }
        },10)
      })
      await sleep(30)
    }
  })
}
async function sleep(ms){
  return new Promise((resolve)=>{
    setTimeout(resolve,ms)
  })
}
function readImgWidthHeight(imgUrl){
  return new Promise(resolve => {
    let img = new Image()
    img.crossOrigin = "anonymous"
    img.onload = function (){
      resolve([this.naturalWidth,this.naturalHeight])
    }
    img.src = imgUrl
  })
}
</script>

<template>
<div class="parent" :style="{
  background:showImg?'unset':'black'
}">
  <div style="position: relative;max-width: 100%;max-height: 100%;" :style="{
  aspectRatio: `${imgWidth}/${imgHeight}`
}">
    <svg :width="imgWidth" :height="imgHeight" version="1.1"
         :viewBox="`0 0 ${imgWidth} ${imgHeight}`"
         id="svg310"
         style="width: 100%;height: 100%;display: block"
         sodipodi:docname="cover_98698741_p0.svg"
         inkscape:version="1.2 (dc2aedaf03, 2022-05-15)"
         xmlns:inkscape="http://www.inkscape.org/namespaces/inkscape"
         xmlns:sodipodi="http://sodipodi.sourceforge.net/DTD/sodipodi-0.dtd"
         xmlns="http://www.w3.org/2000/svg"
         xmlns:svg="http://www.w3.org/2000/svg">
      <g
          fill="none"
          stroke="#000000"
          stroke-linejoin="round"
          id="g308">
        <path v-for="item in v" :d="item.d" :id="`path${item.index}`" stroke-width="2.5" :style="{
        strokeDasharray: item.length,
        strokeDashoffset: item.offset,
        stroke: item.color
      }" v-show="item.length"></path>
      </g>
    </svg>
    <div style="position: absolute;top: 0;left: 0;width: 100%;" class="img" :class="showImg?'opaKey':''">
      <img :src="imgUrl" width="100%" ref="imgIns"/>
    </div>
  </div>
</div>
</template>
<style>
body{
  overflow: hidden;
}
</style>
<style scoped>
div{
  transition: width 3s ease,height 3s ease,background-color .3s ease;
}
path{
  transition: stroke .3s ease,stroke-dashoffset .3s ease;
}
.img{
  opacity: 0;
}
.opaKey{
  animation-name: opaIn;
  animation-duration: .3s;
  animation-fill-mode: forwards;
}
@keyframes opaIn {
  from{
    opacity: 0;
  }
  to{
    opacity: 1;
  }
}
.parent{
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
