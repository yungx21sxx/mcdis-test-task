<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import UiButton from './UIButton.vue'

interface FileInputProps {
  label?: string
  hint?: string
  error?: string | null
  required?: boolean
  disabled?: boolean
  id?: string
  uploading?: boolean
}

enum FileInputState {
  EMPTY = 'empty',
  SELECTED = 'selected',
  UPLOADING = 'uploading',
  UPLOADED = 'uploaded'
}

const props = withDefaults(defineProps<FileInputProps>(), {
  label: 'Label',
  hint: 'Hint text',
  error: null,
  required: false,
  disabled: false,
  id: '',
  uploading: false
})

const emit = defineEmits<{
  (e: 'update:files', files: File[]): void
  (e: 'error', message: string): void
  (e: 'cancel'): void
  (e: 'delete', file: File): void
  (e: 'update:uploading', uploading: boolean): void
}>()

const fileInputRef = ref<HTMLInputElement | null>(null)
const selectedFiles = ref<File[]>([])
const internalError = ref('')
const isUploaded = ref(false)

const uniqueId = computed(() => props.id || `file-input-${Math.random().toString(36).substring(2, 9)}`)

const buttonVariant = computed(() => {
  if (props.disabled) return 'DISABLED'
  if (internalError.value || props.error) return 'FOCUSED'
  return 'DEFAULT'
})

const errorMessage = computed(() => internalError.value || props.error)

const currentState = computed(() => {
  if (selectedFiles.value.length === 0) {
    return FileInputState.EMPTY
  }
  
  if (props.uploading) {
    return FileInputState.UPLOADING
  }
  
  if (isUploaded.value) {
    return FileInputState.UPLOADED
  }
  
  return FileInputState.SELECTED
})

const buttonText = computed(() => {
  const state = currentState.value
  
  switch (state) {
    case FileInputState.EMPTY:
      return 'Выбрать файл'
    case FileInputState.UPLOADING:
      return 'Отменить загрузку'
    case FileInputState.UPLOADED:
      return 'Удалить'
    case FileInputState.SELECTED:
      return 'Выбрать файл'
    default:
      return 'Выбрать файл'
  }
})

const handleButtonClick = () => {
  const state = currentState.value
  
  switch (state) {
    case FileInputState.EMPTY:
    case FileInputState.SELECTED:
      triggerFileInput()
      break
    case FileInputState.UPLOADING:
      cancelUpload()
      break
    case FileInputState.UPLOADED:
      if (selectedFiles.value.length > 0) {
        handleDelete(selectedFiles.value[0])
      }
      break
    default:
      break
  }
}

const handleFileChange = (event: Event) => {
  const input = event.target as HTMLInputElement
  if (!input.files?.length) {
    return
  }
  
  const files = Array.from(input.files)
  
  internalError.value = ''
  isUploaded.value = false
  
  selectedFiles.value = [files[0]]
  
  emit('update:files', selectedFiles.value)
}

const cancelUpload = () => {
  emit('cancel')
  emit('update:uploading', false)
}

const handleDelete = (file: File) => {
  selectedFiles.value = []
  isUploaded.value = false
  emit('delete', file)
  emit('update:files', [])
  
  if (fileInputRef.value) {
    fileInputRef.value.value = ''
  }
}

const triggerFileInput = () => {
  if (!props.disabled && fileInputRef.value) {
    fileInputRef.value.click()
  }
}

watch(() => props.uploading, (newValue, oldValue) => {
  if (oldValue === true && newValue === false && selectedFiles.value.length > 0) {
    isUploaded.value = true
  }
})
</script>

<template>
  <div class="file-input" :class="{ 'file-input--error': errorMessage, 'file-input--disabled': disabled }">
    <label :for="uniqueId" class="file-input__label">
      {{ label }}
      <span v-if="required" class="file-input__required" aria-hidden="true">*</span>
    </label>
    
    <div class="file-input__container">
      <input
        ref="fileInputRef"
        type="file"
        :id="uniqueId"
        class="file-input__input"
        :disabled="disabled"
        :required="required"
        :aria-invalid="!!errorMessage"
        :aria-describedby="`${uniqueId}-hint ${uniqueId}-error`"
        @change="handleFileChange"
      />

      <div class="file-input__main-section">

        <UiButton
          :variant="buttonVariant"
          @click="handleButtonClick"
          :aria-disabled="disabled"
        >
          {{ buttonText }}
        </UiButton>


        <div v-if="selectedFiles.length > 0" class="file-input__file-info">
          <svg v-if="uploading" class="file-input__spinner-icon" xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24"><path fill="currentColor" d="M12,4a8,8,0,0,1,7.89,6.7A1.53,1.53,0,0,0,21.38,12h0a1.5,1.5,0,0,0,1.48-1.75,11,11,0,0,0-21.72,0A1.5,1.5,0,0,0,2.62,12h0a1.53,1.53,0,0,0,1.49-1.3A8,8,0,0,1,12,4Z"><animateTransform attributeName="transform" dur="0.75s" repeatCount="indefinite" type="rotate" values="0 12 12;360 12 12"/></path></svg>
          <span class="file-input__file-name">{{ selectedFiles[0].name }}</span>
        </div>
        <div v-else class="file-input__file-info">
          <span class="file-input__file-placeholder">Файл не выбран</span>
        </div>
      </div>
    </div>
    
    <div v-if="hint && !errorMessage" :id="`${uniqueId}-hint`" class="file-input__hint">
      {{ hint }}
    </div>
    
    <div v-if="errorMessage" :id="`${uniqueId}-error`" class="file-input__error" role="alert">
      {{ errorMessage }}
    </div>
  </div>
</template>

<style scoped lang="sass">
@use '@/assets/variables' as *

.file-input
  display: flex
  flex-direction: column
  width: 100%

  &__label
    font-size: $text-sm
    color: $web-neutral-900
    line-height: 120%
    margin-bottom: 12px
    cursor: pointer
  
  &__required
    color: $web-red-500
    margin-left: 2px
  
  &__container
    display: flex
    flex-direction: column
    gap: 8px
  
  &__input
    position: absolute
    width: 1px
    height: 1px
    padding: 0
    margin: -1px
    overflow: hidden
    clip: rect(0, 0, 0, 0)
    white-space: nowrap
    border-width: 0

  &__spinner-icon
    color: $web-primary-500
  
  &__main-section
    display: flex
    align-items: center
    gap: 16px
  
  &__file-info
    display: flex
    align-items: center
    gap: 8px
    overflow: hidden
    flex-grow: 1
  
  &__file-name
    font-size: $text-md
    color: $web-neutral-700
    white-space: nowrap
    overflow: hidden
    text-overflow: ellipsis
  
  &__file-placeholder
    font-size: $text-md
    color: $web-neutral-400
    white-space: nowrap
    overflow: hidden
    text-overflow: ellipsis
  
  &__hint
    font-size: $text-sm
    color: $web-neutral-500
    margin-top: 8px
  
  &__error
    font-size: $text-sm
    color: $web-red-500
    margin-top: 8px
  
  &--disabled
    opacity: 0.7
    pointer-events: none
    
    .file-input__label
      cursor: not-allowed

</style>