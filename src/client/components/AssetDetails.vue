<script setup lang="ts">
import type { AssetInfo } from '../../types'
import { rpc } from '../logic/rpc'

const props = defineProps<{
  asset: AssetInfo
}>()

const origin = window.parent.location.origin

const imageMeta = computedAsync(() => {
  if (props.asset.type !== 'image')
    return undefined
  return rpc.getImageMeta(props.asset.filePath)
})

const textContent = computedAsync(() => {
  if (props.asset.type !== 'text')
    return undefined
  return rpc.getTextAssetContent(props.asset.filePath)
})

// const copy = useCopy()
const timeago = useTimeAgo(() => props.asset.mtime)
const fileSize = computed(() => {
  const size = props.asset.size
  if (size < 1024)
    return `${size} B`
  if (size < 1024 * 1024)
    return `${(size / 1024).toFixed(2)} KB`
  return `${(size / 1024 / 1024).toFixed(2)} MB`
})

const aspectRatio = computed(() => {
  if (!imageMeta.value?.width || !imageMeta.value?.height)
    return ''
  const gcd = (a: number, b: number): number => {
    if (!b)
      return a
    return gcd(b, a % b)
  }
  const ratio = gcd(imageMeta.value.width, imageMeta.value.height)
  if (ratio > 3)
    return `${imageMeta.value.width / ratio}:${imageMeta.value.height / ratio}`
  return ''
})

const supportsPreview = computed(() => {
  return [
    'image',
    'text',
    'video',
    'font',
  ].includes(props.asset.type)
})
</script>

<template>
  <div flex="~ col gap-4" min-h-full w-full of-hidden p4>
    <template v-if="supportsPreview">
      <div flex="~ gap2" mb--2 items-center op50>
        <div x-divider />
        <div flex-none>
          Preview
        </div>
        <div x-divider />
      </div>

      <div flex="~" items-center justify-center>
        <AssetPreview
          max-h-80 min-h-20 min-w-20 w-auto rounded border="~ base"
          :asset="asset"
          :text-content="textContent"
        />
      </div>
    </template>

    <div flex="~ gap2" mb--2 items-center op50>
      <div x-divider />
      <div flex-none>
        Details
      </div>
      <div x-divider />
    </div>

    <table max-w-full w-full table-fixed>
      <tbody>
        <tr>
          <td w-30 ws-nowrap pr5 text-right op50>
            Filepath
          </td>
          <td>
            <div flex="~ gap-1" w-full items-center>
              <FilepathItem :filepath="asset.filePath" text-left />
              <!-- <VIconButton
                flex-none
                title="Open in Editor"
                icon="carbon-launch"
                @click="openInEditor(asset.filePath)"
              /> -->
            </div>
          </td>
        </tr>
        <tr>
          <td w-30 ws-nowrap pr5 text-right op50>
            Public Path
          </td>
          <td>
            <div flex="~ gap-1" w-full items-center of-hidden>
              <div flex-auto of-hidden truncate ws-pre font-mono>
                {{ asset.publicPath }}
              </div>
              <!-- <VIconButton
                flex-none
                title="Copy public path"
                icon="carbon-copy"
                @click="copy(asset.publicPath)"
              /> -->
              <VIconButton
                flex-none
                :to="`${origin}${asset.publicPath}`"
                icon="carbon-launch"
                target="_blank"
                title="Open in browser"
              />
            </div>
          </td>
        </tr>
        <tr>
          <td w-30 ws-nowrap pr5 text-right op50>
            Type
          </td>
          <td capitalize>
            {{ asset.type }}
          </td>
        </tr>
        <template v-if="imageMeta?.width">
          <tr>
            <td w-30 ws-nowrap pr5 text-right op50>
              Image Size
            </td>
            <td>{{ imageMeta.width }} x {{ imageMeta.height }}</td>
          </tr>
          <tr v-if="aspectRatio">
            <td w-30 ws-nowrap pr5 text-right op50>
              Aspect Ratio
            </td>
            <td>{{ aspectRatio }}</td>
          </tr>
        </template>
        <tr>
          <td w-30 ws-nowrap pr5 text-right op50>
            File size
          </td>
          <td>{{ fileSize }}</td>
        </tr>
        <tr>
          <td w-30 ws-nowrap pr5 text-right op50>
            Last modified
          </td>
          <td>{{ new Date(asset.mtime).toLocaleString() }} <span op70>({{ timeago }})</span></td>
        </tr>
      </tbody>
    </table>

    <div flex="~ gap2" mb--2 items-center op50>
      <div x-divider />
      <div flex-none>
        Actions
      </div>
      <div x-divider />
    </div>
    <div flex="~ gap2 wrap">
      <VButton :to="`${origin}${asset.publicPath}`" download target="_blank" icon="carbon-download">
        Download
      </VButton>
    </div>

    <div flex-auto />
  </div>
</template>
