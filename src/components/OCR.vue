<template>
   
   <section class="wrapper">
      <h1>Extract text from image</h1>
      <small>
         Upload your image here to extract text, this tool built using Tesseract JS
         <br />
       </small>
       
      <div class="input-wrapper" v-if="showUpload">
         <label for="file" class="custom-file-input">Upload</label>
         <input id="file" type="file" @change="getFile" />
         <p>Support format file .jpg .jpeg .png .bmp. Upload high resolution for better result</p>
      </div>
      
      <div class="preview-wrapper" v-if="showPreview">
         <div class="preview">
            <img ref="img" :src="src" width="240"  />
         </div>
         
         <div class="progress-wrapper">
            <p class="progress" ref="status"></p>
         </div>
         <br />
         <button :disabled="isLoad" class="btn-action" @click="recognize" type="button">Recognize</button>
      </div>
      
      
      <div class="result-wrapper" v-if="showResult">
         <div class="result">
            <h4>Result :</h4>
            <pre> {{ result }} </pre>
         </div>
         <button class="btn-close" @click="btnClose" type="button">Close</button>
      </div>
      
   </section>
   <footer>
      Developed by <a href="http://github.com/or-abdillh">Oka R. Abdillh</a>
   </footer>
</template>

<style scoped lang="scss">
   
   @import '../style.scss'
   
</style>

<script setup>

   import { ref, watch } from 'vue'
   import { createWorker, PSM, OEM } from 'tesseract.js';
   
   //
   const showPreview = ref(false)
   const showResult = ref(false)
   const showProgress = ref(false)
   const showUpload = ref(true)
   const isLoad = ref(false)
   
   //Preview image and get a file
   const src = ref('')
   
   //Show result
   const result = ref('')
   const img = ref(null)
   const status = ref('')
   
   //Btn close
   const btnClose = () => {
      showResult.value = false
      showUpload.value = true
      src.value = ''
      status.value.innerHTML = ''
   }

   const getFile = e => {
      
      //Init reader
      const reader = new FileReader()
      reader.onload = () => {
         src.value = reader.result  
      }
      reader.readAsDataURL(e.target.files[0])
      showPreview.value = true
      showUpload.value = false
      
   }
   
   const worker = createWorker({
     logger: m => status.value.innerHTML = `${m.status} : ${(m.progress * 100).toFixed(2)}%`
   })
   
   const recognize = async () => {
      
      isLoad.value = true
      await worker.load()
      await worker.loadLanguage('eng')
      await worker.initialize('eng', OEM.LSTM_ONLY)
      
      await worker.setParameters({
        tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
      })
      
      const { data } = await worker.recognize(img.value)
      result.value = data.text
      showResult.value = true
      showPreview.value = false
      status.value.innerHTML = ''
      isLoad.value = false
   } 
   
</script>

