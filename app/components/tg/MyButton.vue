<template>
  <button 
    :class="['btn', variant, sizeClass, { 'btn-disabled': disabled, 'loading': loading, 'btn-block': block }]" 
    v-bind="$attrs"
    @click="handleClick"
  >
    <slot />
  </button>
</template>

<script setup>
const props = defineProps({
  variant: { type: String, default: 'btn-primary' },  // btn-secondary/outlined 等
  size: { type: String, default: 'btn-md' },
  loading: Boolean,
  disabled: Boolean,
  block: Boolean,
})

const haptic = useHapticFeedback()

const handleClick = (e) => {
  if (props.loading || props.disabled) return
  haptic.selectionChanged()  // 或 props.haptic 控制
  emit('click', e)
}
</script>