<template>
  <view :style="{height: height, width: width, backgroundColor: bgColor}" class="my-2" />
</template>

<script>
export default {
  props: {
    bgColor: {
      type: String,
      default: '#E4E4E4'
    },
    width: {
      type: String,
      default: '750rpx'
    },
    height: {
      type: String,
      default: '2rpx'
    }
  },
  data() {
    return {}
  },
  methods: {}
}
</script>
<style lang="scss">
</style>
