<template>
  <div>
    <h1>Stable Diffusion Image Splitter</h1>
    <p>Select a file using the default filename from the DreamBot Discord message</p>
    <input type="file" @change="readImage" accept="image/*">
    <div class="image-list">
      <img
        v-for="(item, index) in imageResults" :key="index"
        :src="item"
      >
    </div>
  </div>
</template>

<script setup lang="ts">
const imageResults = ref([])

function readImage (e: Event) {
  const file = (e.target as HTMLInputElement).files[0]
  if (file) {
    const reader = new FileReader()
    reader.readAsDataURL(file)
    reader.addEventListener('load', () => {
      let numberOfImages = 1
      const splitInfo = {
        columns: 0,
        rows: 0
      }

      const fileSplit = file.name.split('_')
      if (fileSplit.indexOf('-n') !== -1) {
        const parsedInt = parseInt(fileSplit[fileSplit.indexOf('-n') + 1])
        if (!isNaN(parsedInt)) {
          numberOfImages = parsedInt
        }
      }

      switch(numberOfImages) {
        case 1:
          splitInfo.columns = 1
          splitInfo.rows = 1
          break
        case 2:
          splitInfo.columns = 2
          splitInfo.rows = 1
          break
        case 3:
          splitInfo.columns = 3
          splitInfo.rows = 1
          break
        case 4:
          splitInfo.columns = 2
          splitInfo.rows = 2
          break
        case 5:
          splitInfo.columns = 2
          splitInfo.rows = 3
          break
        case 6:
          splitInfo.columns = 3
          splitInfo.rows = 2
          break
        case 7:
          splitInfo.columns = 3
          splitInfo.rows = 3
          break
        case 8:
          splitInfo.columns = 4
          splitInfo.rows = 2
          break
        case 9:
          splitInfo.columns = 3
          splitInfo.rows = 3
          break
      }

      console.log(`Splitting image into ${numberOfImages} images using columns: ${splitInfo.columns}, rows: ${splitInfo.rows}`)
      splitImage(reader.result as string, splitInfo)
    })
  }
}

function splitImage (imageSrc: string, splitInfo: { columns: number, rows: number }) {
  imageResults.value = []
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')

  const image = new Image()
  image.crossOrigin = 'Anonymous'
  image.src = imageSrc
  image.addEventListener('load', () => {

    const cropWidth = image.width / splitInfo.columns
    const cropHeight = image.height / splitInfo.rows
    canvas.width = cropWidth
    canvas.height = cropHeight

    for (let y = 0; y < splitInfo.rows; ++y) {
      for (let x = 0; x < splitInfo.columns; ++x) {
        ctx.drawImage(image, x * cropWidth, y * cropHeight, cropWidth, cropHeight, 0, 0, canvas.width, canvas.height)
        imageResults.value.push(canvas.toDataURL())
      }
    }
  })
}
</script>

<style>
body {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  text-align: center;
  background-color: #13161c;
  color: #eee;
}

.image-list {
  padding: 1rem 0;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 8px;
}

.image-list img {
  max-width: 512px;
}
</style>