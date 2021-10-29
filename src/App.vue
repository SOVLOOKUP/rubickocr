<script setup lang="ts">
import { createWorker } from 'tesseract.js'
import { ref } from 'vue'

const worker = createWorker({
  langPath: `${document.baseURI}model`,
  logger: m => console.log(m),
})

const out = ref("")
const status = ref({
  fileName: "",
  img: "",
  ready: false
})

const loadModel = async () => {
  await worker.load()
  // 中英文识别
  await worker.loadLanguage('eng+chi_sim')
  status.value.ready = true
}

const recognize = async (ch: boolean) => {
  await worker.initialize(ch ? 'chi_sim' : 'eng')
  const { data: { text } } = await worker.recognize(status.value.img)
  out.value = text
}

const handleFileChange = (event: any) => {
  const file = event.files[0]
  status.value.fileName = file.name
  let reader = new FileReader()
  reader.readAsDataURL(file)
  reader.onload = (e) => {
    if (e.target !== null) status.value.img = e.target.result as string
  }
}

loadModel()

</script>

<template>
  <h1>OCR插件</h1>

  <div
    class="dropzone"
    @dragover.prevent
    @dragenter.prevent
    @dragstart.prevent
    @drop.prevent="handleFileChange($event.dataTransfer)"
  >
    <input
      id="file-input"
      type="file"
      accept="image/png, image/jpeg"
      @change="handleFileChange($event.target)"
      required
    />
    <h3 v-if="status.img">文件名称: {{ status.fileName }}</h3>
    <h2 v-else for="file-input">点击或拖拽图片到此</h2>
    <img v-if="status.img" :src="status.img" />
  </div>

  <div v-if="status.ready">
    <button v-if="status.img" @click="() => recognize(false)">识别上图(英)</button>
    <button v-if="status.img" @click="() => recognize(true)">识别上图(中)</button>
  </div>
  
  <div v-else>
    模型加载中...
  </div>

  <p>{{ out }}</p>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  flex-direction: column;
}
.dropzone {
  height: fit-content;
  min-height: 200px;
  max-height: 400px;
  width: 600px;
  background: #fdfdfd;
  border-radius: 5px;
  border: 2px dashed #000;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 0 auto;
}
input[type="file"] {
  position: absolute;
  opacity: 0;
  width: inherit;
  min-height: 200px;
  max-height: 400px;
  cursor: pointer;
}
img {
  width: 50%;
  height: 50%;
}
button {
  background-color: transparent;
  border: 2px solid #e74c3c;
  border-radius: 1em;
  color: #e74c3c;
  cursor: pointer;
  align-self: center;
  font-size: 1rem;
  margin: 20px;
  padding: 1.2em 2.4em;
  text-align: center;
  text-transform: uppercase;
  font-family: "Montserrat", sans-serif;
  font-weight: 700;
}
</style>