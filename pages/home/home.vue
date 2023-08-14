<template>
  <view>
    <!-- 使用自定义的搜索组件 -->
    <view class="search-box">
      <my-search  @click="gotoSearch"></my-search>
    </view>
    
    <!-- 轮播图的区域 -->
    <swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" circular="true">
      <swiper-item v-for="(item, i) in swiperList" :key="i">
        <navigator class="swiper-item" :url="'/subpkg/goods_detail/goods_detail?goods_id = ' + item.goods_id">
          <image :src="item.image_src" mode=""></image>
        </navigator>
      </swiper-item>
    </swiper>
    
    <!-- 分类导航的区域 -->
    <view class="nav-list">
      <view class="nav-item" v-for="(item, i) in navList" :key="i" @click="navClickHandler(item)">
        <image :src="item.image_src" mode="" class="nav-img"></image>
      </view>
    </view>
    
    <!-- 楼层区域 -->
    
    <!-- 楼层的容器 -->
    <view class="floor-list">
      
      <!-- 每一个楼层的 item 项 -->
      <view class="floor-item" v-for="(item, i) in floorList" :key="i">
        
        <!-- 楼层的标题 -->
        <image :src="item.floor_title.image_src" mode="" class="floor-title"></image>
        
        <!-- 楼层的图片区域 -->
        <view class="floor-img-box">
          
          <!-- 左侧大图片的盒子 -->
          <navigator class="left-img-box" :url="item.product_list[0].url">
            <image :src="item.product_list[0].image_src" :style="{width: item.product_list[0].image_width + 'rpx'}" mode="widthFix"></image>
          </navigator>
          
          <!-- 右侧小图片的盒子 -->
          <view class="right-img-box">
            <navigator class="right-img-item" v-for="(item, i) in item.product_list" :key="i" v-if="i !== 0" :url="item.url">
              <image :src="item.image_src" :style="{width: item.image_width + 'rpx'}" mode="widthFix"></image>
            </navigator>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        //这是轮播图的数据列表
        swiperList: [],
        //分类导航的数据列表
        navList: [],
        //楼层的数据
        floorList: []
      };
    },
    onLoad() {
      //调用方法获取轮播图的数据
      this.getSwiperList()
      //调用方法获取导航栏的数据
      this.getNavList()
      //调用方法获取楼层的数据
      this.getFloorList()
    },
    methods: {
      //获取轮播图数据
      async getSwiperList() {
        const {data:res} = await uni.$http.get('/api/public/v1/home/swiperdata')
        //请求失败
        if(res.meta.status !== 200) return uni.$showMsg()
        
        //请求成功
        this.swiperList = res.message
      },
      //获取导航栏数据
      async getNavList() {
        const {data:res} = await uni.$http.get('/api/public/v1/home/catitems')
        if(res.meta.status !== 200) return uni.$showMsg()
        this.navList = res.message
      },
      //点击事件--跳转至分类页面
      navClickHandler(item) {
        if(item.name === '分类') {
          uni.switchTab({
            url: '/pages/cate/cate'
          })
        }
      },
      //获取首页楼层数据
      async getFloorList() {
        const {data:res} = await uni.$http.get('/api/public/v1/home/floordata')
        if(res.meta.status !== 200) return uni.$showMsg()
        //对数据进行处理
        res.message.forEach(floor => {
          floor.product_list.forEach(prod => {
            prod.url = '/subpkg/goods_list/goods_list?' + prod.navigator_url.split('?')[1]
          })
        })
        this.floorList = res.message
      },
      
      gotoSearch() {
        console.log('ok')
        // uni.navigateTo({
        //   url: '/subpkg/search/search'
        // })
      }
    }
  }
</script>

<style lang="scss">
  .search-box {
    //设置定位效果为 "吸顶"
    position: sticky;
    //"吸顶" 的位置
    top: 0;
    //提高层级,防止被轮播图覆盖
    z-index: 999;
  }
  
swiper {
  height: 330rpx;
  .swiper-item,
  image {
    width: 100%;
    height: 100%;
  }
}

.nav-list {
  display: flex;
  justify-content: space-around;
  margin: 15px 0;
  .nav-img {
    width: 128rpx;
    height: 140rpx;
  }
}

.floor-title {
  width: 100%;
  height: 60rpx;
}

.floor-img-box {
  display: flex;
  padding-left: 10rpx;
}

.right-img-box {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}
</style>
