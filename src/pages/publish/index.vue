<template>
  <!-- 组件必须设置高度，否则内部容器填充不起来 -->
  <view class="w-full overflow-auto album-content">
    <lz-album
      ref="rAlbum"
      v-model="list"
      :address="address"
      :location="location"
      :lon_lat="lon_lat"
      :weather="weather"
      @add="onAlbumAdd"
      @click="onAlbumClick"
      @delete="onAlbumDelete"
      @sort="onAlbumSort"
    >
      <template #before>
        <view class="px-3 pt-2">
          <u-input
            v-model="content"
            :auto-height="false"
            :focus="false"
            :maxlength="927"
            class="moment-think"
            height="50"
            placeholder="这一刻的想法..."
            type="textarea"
          />
        </view>
      </template>
      <template #after>
        <view class="px-2 pb-2">
          <u-cell-group>
            <u-cell-item :arrow="false" icon="star" title="萧潇提示" value="可选择36张图片，一次最多9张" />
          </u-cell-group>
          <u-cell-group>
            <u-cell-item :value="location" icon="map" title="所在位置" @click="chooseLocation" />
          </u-cell-group>
          <u-cell-group>
            <u-cell-item :arrow="false" :value="weather" icon="heart" title="当前天气" />
          </u-cell-group>
          <u-button :loading="loading" open-type="getUserProfile" ripple type="success" @click="prePublish">发表</u-button>
        </view>
      </template>
    </lz-album>
  </view>
</template>

<script>
import LzAlbum from '@/components/lz-album'
import { dbRequest, getWeather } from '@/api/common'
import { getLocation, mapSearch, reverseGeocoder } from '@/utils'

export default {
  components: {
    LzAlbum
  },
  data() {
    return {
      // id是DB主键，如果id重复会造成在拖拽排序时候，会影响相同ID的元素
      list: [],
      content: '',
      location: '深圳市',
      address: '', // 完整地址

      loading: false,
      userInfo: null,
      openid: '',
      weather: '',
      lon_lat: {}
    }
  },
  onLoad() {
    this.openid = uni.getStorageSync('openid')
    this.userInfo = uni.getStorageSync('userInfo')
    this.chooseLocation()
  },
  onShow() {
    // this.getLocation()
  },
  methods: {
    getLocation() {
      getLocation().then(res => {
        const { longitude, latitude } = res
        this.lon_lat = { longitude, latitude }
        this.getLocationInfo({ longitude, latitude })
        // this.getNearbyAttractions(longitude, latitude)
        this.getWeather(latitude + ':' + longitude)
      })
    },
    chooseLocation() {
      wx.chooseLocation({})
        .then(res => {
          console.log(res)
          const { longitude, latitude, name, address } = res
          this.lon_lat = { longitude, latitude }
          this.address = address + name
          this.getLocationInfo({ longitude, latitude })
          this.getWeather(latitude + ':' + longitude)
        })
        .catch(() => {
          this.getLocation()
        })
    },
    getLocationInfo(location) {
      reverseGeocoder(location).then(res => {
        console.log('当前位置信息：', res)
        const { province, city, district } = res.result.ad_info
        this.location = province + city + district
      })
    },
    getNearbyAttractions(longitude, latitude) {
      mapSearch('社区', {
        longitude,
        latitude
      }).then(res => {
        console.log(res)
      })
    },
    getWeather(area) {
      getWeather(area).then(([error, res]) => {
        const weatherInfo = res.data.results[0].now
        this.weather = weatherInfo.text + '：' + weatherInfo.temperature + '°'
        console.log(error)
      })
    },
    queryOpenid(openid) {
      const actions = [
        {
          method: 'where',
          params: {
            openid
          }
        },
        { method: 'get' }
      ]
      return dbRequest('user', actions)
    },
    addUser(openid, userInfo) {
      const actions = [
        {
          method: 'add',
          params: {
            data: {
              openid,
              userInfo
            }
          }
        }
      ]
      dbRequest('user', actions).then(res => {
        console.log(res)
      })
    },
    prePublish() {
      if (this.userInfo) {
        this.publish()
        return
      }
      wx.getUserProfile({
        desc: '完善用户信息',
        success: res => {
          const userInfo = res.userInfo
          const openid = this.openid
          uni.setStorageSync('userInfo', userInfo)
          this.userInfo = userInfo
          this.queryOpenid(openid).then(res => {
            if (!res.data.length) {
              this.addUser(openid, userInfo)
            }
            this.publish()
          })
        }
      })
    },
    publish() {
      const imgList = []
      const list = this.$refs.rAlbum.list
      list.forEach(media => {
        console.log(media)
        const { lon_lat, location, address, weather } = this
        if (media.cloudPath) {
          const {
            cloudPath,
            fileType,
            height,
            id,
            size,
            sortID,
            width,
            thumbCloudPath
          } = media
          imgList.push({
            cloudPath,
            fileType,
            height,
            id,
            size,
            sortID,
            width,
            thumbCloudPath,
            lon_lat,
            location,
            address,
            weather
          })
        }
      })
      if (!imgList.length && !this.content) {
        uni.showToast({ title: '请输入内容或上传', icon: 'none' })
        return
      }
      const actions = [
        {
          method: 'add',
          params: {
            data: {
              openid: this.openid,
              content: this.content,
              location: this.location,
              address: this.address,
              publishTime: Date.now(),
              imgList,
              name: this.userInfo.nickName,
              avatar: this.userInfo.avatarUrl,
              lon_lat: this.lon_lat,
              weather: this.weather
            }
          }
        }
      ]
      dbRequest('blog', actions).then(res => {
        this.reset()
        uni.switchTab({ url: '/pages/index/index' })
      })
    },
    reset() {
      this.$refs.rAlbum.reset()
      this.$refs.rAlbum.init()
      this.$refs.rAlbum.initGrid()
      this.content = ''
    },
    onAlbumSort(list) {
      // 返回排序后的数组集合
      // 更新后台排序号
      console.info('排序成功')
    },
    onAlbumClick(res) {
      // 图片预览
      uni.previewImage(res.index, res.list)
    },
    onAlbumAdd(res) {
      // 新增图片，会返回图片的排序号和图片路径
      // 需要保存到后台后，然后add到组件中
      console.info('新增成功')
      console.log(res)
      this.$refs.rAlbum.add({
        id: res.cloudPath,
        sortID: res.sortID,
        src: res.src,
        ...res
      })
    },
    onAlbumDelete(data) {
      // 返回删除的图片实体
      // 到后台删除
      console.info('删除成功')
    }
  }
}
</script>

<style lang="scss">
.moment-think {
  ::v-deep .u-input__input {
    height: 300upx;
    font-size: 16px;
  }
}
</style>
