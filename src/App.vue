<script setup>
import {ref, watch} from 'vue'
import UiButton from "@/components/UI/UIButton.vue";
import UIFileInput from "@/components/UI/UIFileInput.vue";
import UIButton from "@/components/UI/UIButton.vue";

const selectedFile = ref([])
const isUploading = ref(false)
const fileWithError = ref([])

const handleFilesSelected = (files) => {
  console.log('File selected:', files)
}

const handleError = (message) => {
  console.error('Error:', message)
}

let abortController = null

const simulateUpload = async () => {
  if (selectedFile.value.length > 0) {
    abortController = new AbortController()
    const signal = abortController.signal
    
    isUploading.value = true
    try {
      await new Promise((resolve, reject) => {
        const startTime = Date.now()
        const duration = 2000
        
        const updateProgress = () => {
          if (signal.aborted) return
          
          const elapsed = Date.now() - startTime
          if (elapsed < duration && !signal.aborted) {
            requestAnimationFrame(updateProgress)
          }
        }
        
        updateProgress()
        
        const timerId = setTimeout(() => {
          if (!signal.aborted) {
            resolve('Загрузка завершена')
          }
        }, duration)
        
        signal.addEventListener('abort', () => {
          clearTimeout(timerId)
          reject(new Error('Операция отменена'))
        })
      })
      
      console.log('Файл успешно загружен:', selectedFile.value[0].name)
      isUploading.value = false
    } catch (error) {
      console.log('Операция прервана:', error.message)
      isUploading.value = false
    }
  }
}

watch(selectedFile, () => {
  console.log(selectedFile.value)
})

const handleCancel = () => {
  console.log('Вызвана функция handleCancel')
  
  if (abortController) {
    abortController.abort()
    abortController = null
    console.log('Загрузка отменена пользователем')
  }
  
  isUploading.value = false
}
</script>

<template>
  <div class="container">
    <h1>Компоненты загрузки файлов</h1>
    
    <section>
      <h2>Кнопки загрузки файлов</h2>
      <div class="buttons">
        <div class="buttons__example">
          <span>DEFAULT</span>
          <UiButton variant="DEFAULT">Button</UiButton>
        </div>
        <div class="buttons__example">
          <span>DISABLED</span>
          <UiButton variant="DISABLED">Button</UiButton>
        </div>
        <div class="buttons__example">
          <span>FOCUSED</span>
          <UiButton variant="FOCUSED">Button</UiButton>
        </div>
      </div>
    </section>
    
    <section>
      <h2>Примеры компонента загрузки файлов с FSM</h2>
      
      <div class="file-inputs">
        <div class="file-inputs__example">
          <h3>Демонстрация FSM</h3>
          <UIFileInput
            label="Загрузите файл"
            hint="Выберите файл для загрузки"
            v-model:files="selectedFile"
            v-model:uploading="isUploading"
            @error="handleError"
            @cancel="handleCancel"
          />
          
          <div class="file-inputs__actions">
            <UIButton
              class="upload-button" 
              @click="simulateUpload"
              :variant="(!selectedFile.length || isUploading) ? 'DISABLED' : 'DEFAULT'"
            >
              Имитировать загрузку файла
            </UIButton>
          </div>
          <p>Загрузка длится 2 секунды</p>
        </div>
        
        <div class="file-inputs__example">
          <h3>С ошибкой</h3>
          <UIFileInput
            label="Загрузите файл"
            hint="Выберите файл для загрузки"
            error="Произошла ошибка при загрузке файла"
            v-model:files="fileWithError"
          />
        </div>
        
        <div class="file-inputs__example">
          <h3>Обязательное поле</h3>
          <UIFileInput
            label="Загрузите файл"
            hint="Это поле обязательно для заполнения"
            required
          />
        </div>
        
        <div class="file-inputs__example">
          <h3>Отключенное поле</h3>
          <UIFileInput
            label="Загрузите файл"
            hint="Это поле отключено"
            disabled
          />
        </div>
      </div>
    </section>
  </div>
</template>

<style scoped lang="sass">
@use '@/assets/variables' as *

.container
  h1
    margin-bottom: 32px
    color: $web-neutral-900
  
  h2
    margin-bottom: 24px
    color: $web-neutral-700
  
  h3
    margin-bottom: 16px
    color: $web-neutral-700
    font-size: 16px

section
  margin-bottom: 48px

.buttons
  padding: 32px
  border: 1px solid $web-neutral-300
  border-radius: 16px
  justify-content: space-between
  &, &__example
    display: flex

  &__example
    flex-direction: column
    gap: 16px
    span
      font-size: 12px
      color: $web-neutral-400

.file-inputs
  display: grid
  grid-template-columns: repeat(auto-fill, minmax(400px, 1fr))
  gap: 32px

  @media screen and (max-width: 600px)
    grid-template-columns: 1fr

  &__example
    padding: 24px
    border: 1px solid $web-neutral-300
    border-radius: 16px
    background-color: white
    display: flex
    flex-direction: column
    gap: 16px
  &__actions
    display: flex
    flex-direction: column
    gap: 12px

.upload-button
  display: block
</style>
