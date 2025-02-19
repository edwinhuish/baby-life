<template>
  <view class="w-full" @click="clickDynamic">
    <view class="flex flex-auto overflow-hidden px-3 py-3">
      <view class="user__header-warp">
        <!-- 头像组 -->
        <view class="flex user__header" @click.stop="clickUser">
          <image
            :src="detail.avatar"
            class="layout-abs-center user__header-image"
            mode="aspectFill"
          />
        </view>
      </view>
      <view class="flex flex-auto overflow-hidden user__content">
        <view class="layout-col-slide flex-auto overflow-hidden pl-2">
          <text class="truncate" @click.stop="clickUser()">{{ detail.name }}</text>
          <text class="user__content-note text-gray-400 truncate">
            {{ timestampFormat(detail.publishTime) }}
          </text>
        </view>
        <view class="layout-col-slide">
          <text class="text-gray-400 location">{{ detail.location }}</text>
          <text class="text-gray-400 weather">{{ detail.weather }}</text>
        </view>
      </view>
    </view>

    <view class="text break-words">{{ detail.content }}</view>
    <view class="layout-start-wrap mt-1">
      <view class="imgList">
        <view v-for="(item,index) in imgList" :key="index" class="mr-1 inline-block">
          <lazy-video
            v-if="item.fileType === 'video'"
            :img-height="imgHeight"
            :img-width="imgWidth"
            :item="item"
          />
          <image
            v-else
            :src="item.cloudPath"
            :style="{width:imgWidth+'px','max-height':imgHeight+'px'}"
            lazy-load
            mode="aspectFill"
            @click.stop="previewImg(index)"
          />
        </view>
      </view>
    </view>
    <divider />
  </view>
</template>

<script>
import LazyVideo from '../../components/lazy-video'

export default {
  components: {
    LazyVideo
  },
  props: {
    imgList: {
      default: () => [],
      type: Array
    },
    detail: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      windowWidth: 0, // 屏幕可用宽度
      windowHeight: 0, // 屏幕可用高度
      imgWidth: 0, // 图片宽度
      imgHeight: 0 // 图片高度
    }
  },
  mounted() {
    const res = uni.getSystemInfoSync()
    this.windowHeight = res.windowHeight
    this.windowWidth = res.windowWidth

    this.judgeImg()
  },
  methods: {
    // 预览图片
    previewImg(current) {
      uni.previewImage({
        count: this.imgList[current].cloudPath,
        current,
        urls: this.imgList.map(v => v.cloudPath),
        longPressActions: {
          itemList: ['保存图片']
        }
      })
    },
    // 自适应判断
    judgeImg() {
      if (this.imgList.length === 1) {
        this.imgWidth = this.windowWidth - 30
        this.imgHeight = this.windowHeight * (3 / 5).toFixed(0)
      } else {
        const width = this.windowWidth / 3.4
        this.imgWidth = width.toFixed(0)
        this.imgHeight = width.toFixed(0)
      }
    },
    timestampFormat(timestamp) {
      if (!timestamp) return ''
      const time = this.$dayjs(timestamp)
      const timestampDiff = Date.now() - timestamp // 参数时间戳与当前时间戳相差秒数

      const ret = time.fromNow()
      if (timestampDiff < 60 * 1000) {
        // 一分钟以内
        return '刚刚'
      } else if (timestampDiff < 3600 * 1000) {
        // 一小时前之内
        return ret
      } else if (time.isToday()) {
        return `今天 ${time.format('HH:mm')}`
      } else if (time.isYesterday()) {
        return `昨天 ${time.format('HH:mm')}`
      } else if (new Date().getFullYear().toString() === time.format('YYYY')) {
        return time.format('MM月DD日 HH:mm:ss')
      } else {
        return time.format('YYYY年MM月DD日 HH:mm:ss')
      }
    },

    /** 触发父级事件 */
    // 点击动态
    clickDynamic() {
      this.$emit('clickDynamic')
    },
    // 点击用户信息
    clickUser() {
      this.$emit('clickUser')
    },
    // 点击关注
    clickFocus() {
      this.$emit('clickFocus')
    },
    // 点赞
    clickThumbsup() {
      this.$emit('clickThumbsup')
    },
    // 点击打赏
    clickGiveReward() {
      this.$emit('clickGiveReward')
    },
    // 点击聊天
    clickChat() {
      this.$emit('clickChat')
    }
  }
}
</script>

<style>
.imgList {
  margin: 0 3%;
}
.text {
  margin: 1% 3% 2%;
}
.user__header,
.user__header-image {
  width: 45px;
  height: 45px;
  border: 1px solid #eee;
  overflow: hidden;
  border-radius: 5px;
}
.user__header-image {
  flex-wrap: wrap-reverse;
}
.user__content {
  padding: 2px 0;
}
.user__content-note {
  font-size: 11px;
}
.location,
.weather {
  padding: 3px;
  font-size: 11px;
}
</style>
