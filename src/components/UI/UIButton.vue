<script setup lang="ts">
   const props = defineProps<{
      variant: 'DEFAULT' | 'DISABLED' | 'FOCUSED'
   }>()
</script>

<template>
   <button
      :class="['btn', {
         'btn_default': props.variant === 'DEFAULT',
         'btn_disabled': props.variant === 'DISABLED',
         'btn_focused': props.variant === 'FOCUSED'
      }]"
      :disabled="props.variant === 'DISABLED'"
      type="button"
   >
      <slot />
   </button>
</template>

<style scoped lang="sass">
   @use '@/assets/variables' as *

   .btn
      padding: 12px
      border-radius: 12px
      box-shadow: $shadow-base
      background: #fff
      transition: all .3s ease-out
      color: $web-neutral-700
      cursor: pointer

      &_default, &_disabled
         border: 1px solid $web-neutral-200

      &_disabled
         color: $web-neutral-400
         cursor: default

      &_focused
         border: none
         box-shadow: 0 0 2px 2px rgba(#6B7280, 10%)

      &:hover
         @media (hover: hover)
            background: $web-neutral-100
            &:not(&_focused)
               border: 1px solid $web-neutral-300

      &:active
         background: $web-neutral-200
         &:not(&_focused)
            border: 1px solid $web-neutral-300







</style>