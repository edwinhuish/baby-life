<template>
  <view class="container">
    <u-search v-model="keyword" placeholder="时间、地点、天气..." />
    <view class>
      <u-tabs
        :current="current"
        :is-scroll="false"
        :list="tabList"
        active-color="#fba414"
        height="70"
        inactive-color="#606266"
        @change="changeTab"
      />
    </view>
    <view v-for="(dateItem) in dates" :key="dateItem" class="p-2">
      <view class="pl-2 py-2 layout-slide">
        <view class="text-black font-medium text-base">{{ dateItem }}</view>
        <view class="text-gray-400 text-xs" @click="chooseLocation(imgMapping[dateItem][0])">
          {{ imgMapping[dateItem][0].location }}
          <u-icon name="arrow-right" />
        </view>
      </view>
      <view :class="{video: current === 1}" class="grid-view">
        <view
          v-for="item in imgMapping[dateItem]"
          :key="item.cloudPath"
          class="img-item"
          @click="previewImg(item)"
        >
          <lazy-video
            v-if="item.fileType === 'video'"
            :img-height="120"
            :img-width="150"
            :item="item"
          />
          <u-image v-else :lazy-load="true" :src="item.cloudPath" height="200" width="165" />
        </view>
      </view>
    </view>
    <u-loadmore :load-text="loadText" :status="status" icon-type="flower" />
  </view>
</template>

<script>
import { dbRequest } from '@/api/common'
import { getList } from '@/utils/mixins'
import { fromNowFormat } from '@/utils'
import setting from '@/setting'
import LazyVideo from '../../components/lazy-video'

export default {
  components: {
    LazyVideo
  },
  mixins: [getList],

  data() {
    return {
      tabList: [
        {
          name: '照片',
          type: 'image'
        },
        {
          name: '视频',
          type: 'video'
        },
        {
          name: '图集'
        },
        {
          name: '回忆'
        }
      ],
      current: 0,
      title: 'Hello',
      imgMapping: {},
      dates: [],
      loaded: false,
      previewing: false
    }
  },
  onLoad() {},
  onShow() {
    if (!this.previewing) {
      this.getList(true)
    } else {
      this.previewing = false
    }
  },
  methods: {
    previewImg(data) {
      const index = this.list.findIndex(
        value => value.cloudPath === data.cloudPath
      )
      uni.previewImage({
        count: this.list[index].cloudPath,
        current: index,
        urls: this.list.map(v => v.cloudPath),
        longPressActions: {
          itemList: ['保存图片']
        },
        complete: result => {
          this.previewing = true
        }
      })
    },
    chooseLocation(data) {
      const key = setting.MAP_KEY // 使用在腾讯位置服务申请的key
      const referer = '萧潇生活' // 调用插件的app的名称
      const { latitude, longitude } = data.lon_lat
      const location = JSON.stringify({
        latitude,
        longitude
      })
      wx.navigateTo({
        url:
          'plugin://chooseLocation/index?key=' +
          key +
          '&referer=' +
          referer +
          '&location' +
          location
      })
    },
    changeTab(index) {
      this.current = index
      if (index < 2) {
        this.getList(true)
      }
    },
    getList(isRefresh) {
      uni.showLoading({ title: '获取数据中...' })
      const skipNum = (this.pageNum - 1) * this.pageSize
      const fileType = this.tabList[this.current].type
      const actions = [
        { method: 'orderBy', params: ['createTime', 'desc'] },
        {
          method: 'where',
          params: {
            fileType
          }
        },
        { method: 'get' }
      ]
      if (skipNum) {
        actions.splice(0, 0, { method: 'skip', params: [skipNum] })
      }
      dbRequest('album', actions)
        .then(res => {
          console.log(res)
          this.handleDate(res.data)
          this.handleResFromMixin(res, v => v, isRefresh)
        })
        .catch(() => {
          this.status = 'nomore'
          this.noMore = true
        })
        .finally(() => {
          uni.hideLoading()
          uni.stopPullDownRefresh()
        })
    },
    handleDate(data) {
      const result = {}
      const dates = []
      console.log(data)
      data.forEach(item => {
        const date = fromNowFormat(item.createTime)
        console.log(date)
        if (!result[date]) {
          result[date] = []
          dates.push(date)
        }
        result[date].push(item)
      })
      this.dates = dates
      this.imgMapping = result
      console.log(result)
    }
  },
  onReachBottom() {},
  onPullDownRefresh() {
    this.getList()
  }
}
</script>

<style lang="scss">
.grid-view {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-column-gap: 20upx;
  grid-row-gap: 20upx;
  &.video {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
