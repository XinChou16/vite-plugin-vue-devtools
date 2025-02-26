<script lang="ts">
const iframeCacheMap = new Map<string, HTMLIFrameElement>()
</script>

<script setup lang="ts">
const props = defineProps<{
  src: string
}>()

const colorMode = useColorMode()
const anchor = ref<HTMLDivElement>()
const key = computed(() => props.src)
const iframeEl = ref<HTMLIFrameElement>()
const box = reactive(useElementBounding(anchor))

onMounted(() => {
  if (iframeCacheMap.get(key.value)) {
    iframeEl.value = iframeCacheMap.get(key.value)!
    iframeEl.value.style.visibility = 'visible'
  }
  else {
    iframeEl.value = document.createElement('iframe')
    iframeCacheMap.set(key.value, iframeEl.value)
    iframeEl.value.src = props.src
    // CORS
    try {
      iframeEl.value.style.opacity = '0.01'
      iframeEl.value.onload = () => {
        syncColorMode()
        iframeEl.value!.style.opacity = '1'
      }
    }
    catch (e) {
      iframeEl.value.style.opacity = '1'
    }
    document.body.appendChild(iframeEl.value)
    nextTick(updateIframeBox)
  }
  setTimeout(syncColorMode, 100)
})

watchEffect(updateIframeBox)
watchEffect(syncColorMode)

onUnmounted(() => {
  if (iframeEl.value)
    iframeEl.value.style.visibility = 'hidden'
})

function syncColorMode() {
  if (!iframeEl.value || !iframeEl.value.contentWindow)
    return
  try {
    const html = iframeEl.value.contentWindow.document.querySelector('html')
    html?.classList.toggle('dark', colorMode.value === 'dark')
    html?.classList.toggle('light', colorMode.value === 'dark')
  }
  catch (e) {
  }
}

function updateIframeBox() {
  if (!iframeEl.value)
    return
  Object.assign(iframeEl.value.style, {
    position: 'fixed',
    left: `${box.left}px`,
    top: `${box.top}px`,
    width: `${box.width}px`,
    height: `${box.height}px`,
    outline: 'none',
  })
}
</script>

<template>
  <div ref="anchor" h-full w-full />
</template>
