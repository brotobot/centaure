<script setup>
  import { onMounted } from 'vue'
  import { parseGIF, decompressFrames } from 'gifuct-js'
  const datas = {
    1: {
      src: "1.gif",
      // attack: 1000,
      // rollbackAttack: 0,
      // arrowLeftPosition: {
      //   x: 20,
      //   y: 20,
      // },
      // arrowRightPosition: {
      //   x: 80,
      //   y: 20,
      // },
    }
  }

  const tree = {}

  const imgs = {}

  let actual = 0
  let subdiv = 4
  let interval = null
  let canvas = null
  let ctx = null
  const canvas2 = document.createElement("canvas");
  const ctx2 = canvas2.getContext("2d");
  onMounted(async () => {
    canvas = document.getElementById("mainCanvas")
    ctx = canvas.getContext("2d");
    for (let key in datas) {
      await loadGif(key, datas[key])
    }
    if (interval !== null) clearInterval(interval)
    canvas.width = imgs[1][actual].width
    canvas.height = imgs[1][actual].height
    canvas2.width = imgs[1][actual].width
    canvas2.height = imgs[1][actual].height
    interval = setInterval(() => {
      actual = (actual + (1 / subdiv)) % imgs[1].length
      const act = parseInt(actual)
      ctx2.putImageData(imgs[1][act].datas, imgs[1][act].left, imgs[1][act].top);
      ctx.drawImage(canvas2, 0, 0)
    }, 100)
  })

  function generatePatch(image) {
    var totalPixels = image.pixels.length;
    var patchData = new Uint8ClampedArray(totalPixels * 4);
    for (var i = 0; i < totalPixels; i++) {
      const pos = i * 4;
      const colorIndex = image.pixels[i];
      const color = image.colorTable[colorIndex];
      patchData[pos] = color[0];
      patchData[pos + 1] = color[1];
      patchData[pos + 2] = color[2];
      patchData[pos + 3] = colorIndex !== image.transparentIndex ? 255 : 0;
    }
    return patchData;
  };

async function loadGif(key, img) {
  imgs[key] = []
  return fetch(img.src)
    .then(resp => resp.arrayBuffer())
    .then(buff => {
      var gif = parseGIF(buff)
      var frames = decompressFrames(gif, true)
      for (let f in frames) {
        var frameImageData = ctx.createImageData(frames[f].dims.width, frames[f].dims.height)
        frameImageData.data.set(generatePatch(frames[f]))
        imgs[key][f] = {
          datas: frameImageData,
          top: frames[f].dims.top,
          left: frames[f].dims.left,
          width: frames[f].dims.width,
          height: frames[f].dims.height,
        }
      }
     });
}

</script>

<template>
  <main>
    <canvas id="mainCanvas"></canvas>
    <img src="leftArrow.png" class="leftArrow"/>
    <img src="rightArrow.png" class="rightArrow"/>
  </main>
</template>

<style>
  *{
    padding: 0px;
    margin: 0px;
  }

 #mainCanvas{
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    max-width: 100vw;
    min-height: 100vh;
  }
</style>
