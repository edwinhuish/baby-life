<template>
  <view class="layout-abs-center lazy-video" @click.stop="previewVideo">
    <u-image class="play-icon" height="60" src="/static/icon/play.png" width="60" />
    <image
      v-if="showThumb"
      :src="item.thumbCloudPath"
      :style="{width:imgWidth+'px','max-height':imgHeight+'px'}"
      lazy-load
      mode="aspectFill"
    />
    <video v-else :src="item.cloudPath" :style="{width:imgWidth+'px','max-height':imgHeight+'px'}" />
  </view>
</template>

<script>
export default {
  name: 'Index',
  props: {
    item: {
      type: Object,
      default: () => {}
    },
    imgWidth: {
      type: Number,
      default: 0
    },
    imgHeight: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      showThumb: true
    }
  },
  methods: {
    previewVideo() {
      this.showThumb = false
    }
  }
}
</script>

<style lang="scss" scoped>
.play-icon {
  position: absolute;
}
.lazy-video {
  background: rgba(255, 255, 255, 0.6);
}
</style>
