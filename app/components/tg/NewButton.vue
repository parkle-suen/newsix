<template>
  <component 
    :is="tag" 
    :to="to" 
    :href="href" 
    :class="['btn', variantClass, sizeClass, { 
      'btn-block': block, 
      'btn-disabled': disabled, 
      loading: loading, 
      'btn-glass': elevated,  // elevated → glass 效果（或用 shadow-md 自定义）
      'uppercase': uppercase 
    }, props.class]" 
    :disabled="disabled || loading"
    v-bind="$attrs"
    @click="handleClick"
  >
    <!-- loading 用 DaisyUI 内置 spinner（dots），无需手动 Icon -->
    <!-- icon/title/slot 自由排，位置随意 -->
    <Icon v-if="icon && iconPosition === 'left' && !loading" :name="icon" class="h-5 w-5" />
    <Icon v-if="loading" name="i-heroicons-arrow-path-20-solid" class="h-5 w-5 animate-spin" />  <!-- 可选自定义 spinner -->
    <span v-if="title" :class="{ 'uppercase': uppercase }">{{ title }}</span>
    <slot />
    <Icon v-if="icon && iconPosition === 'right' && !loading" :name="icon" class="h-5 w-5" />
  </component>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { shareURL, useHapticFeedback } from '~/composables/telegram'

const props = withDefaults(defineProps<{
  title?: string
  status?: 'primary' | 'secondary' | 'outline' | 'danger' | 'destructive'  // 映射到 DaisyUI variant
  icon?: string
  iconPosition?: 'left' | 'right'
  to?: string
  href?: string
  shareUrl?: string
  block?: boolean
  small?: boolean  // 兼容旧
  size?: 'xs' | 'sm' | 'md' | 'lg'  // DaisyUI 标准 size
  loading?: boolean
  elevated?: boolean  // 用 glass 或 shadow
  uppercase?: boolean
  disabled?: boolean
  class?: string
  haptic?: boolean | 'selection' | 'impact-light' | 'impact-medium' | 'impact-heavy' | 'notification-success' | 'notification-warning' | 'notification-error'
}>(), {
  status: 'primary',
  iconPosition: 'left',
  block: true,
  small: false,
  size: 'md',
  loading: false,
  elevated: false,
  uppercase: false,
  disabled: false,
  class: '',
  haptic: false,
})

const tag = computed(() => props.to ? 'NuxtLink' : (props.href ? 'a' : 'button'))

const variantClass = computed(() => {
  if (props.status === 'outline') return 'btn-outline'
  if (props.status === 'secondary') return 'btn-neutral'  // 或 btn-ghost/btn-secondary（DaisyUI 有更多）
  if (props.status === 'danger' || props.status === 'destructive') return 'btn-error'
  return 'btn-primary'  // default
})

const sizeClass = computed(() => {
  const size = props.size || (props.small ? 'sm' : 'md')
  return `btn-${size}`  // btn-xs/sm/md/lg
})

const hapticFeedback = useHapticFeedback()

function triggerHaptic() {
  if (!props.haptic) return
  const kind = props.haptic === true ? 'selection' : props.haptic
  try {
    if (kind === 'selection') hapticFeedback.selectionChanged()
    else if (kind.startsWith('impact-')) hapticFeedback.impactOccurred(kind.split('-')[1] as 'light' | 'medium' | 'heavy')
    else if (kind.startsWith('notification-')) hapticFeedback.notificationOccurred(kind.split('-')[1] as 'success' | 'warning' | 'error')
  } catch {
    console.debug('Haptic feedback not supported')
  }  // 静默失败
}

function handleClick(e: Event) {
  if (props.disabled || props.loading) return
  triggerHaptic()
  if (props.shareUrl) {
    e.preventDefault()
    shareURL(props.shareUrl)
  }
}
</script>