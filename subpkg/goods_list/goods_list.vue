<template>
  <view>
    goods_list
  </view>
</template>

<script>
  export default {
    data() {
      return {
        //请求参数对象
        queryObj: {
          query: '',
          cid: '',
          pagenum: 1,
          pagesize: 10,
          goodsList: [],
          total: 0
        },
        
      };
    },
    onLoad(options) {
      this.queryObj.query = options.query || ''
      this.queryObj.cid = options.cid || ''
      //调用获取商品列表数据的方法
      this.getGoodsList()
      
    },
    methods: {
      async getGoodsList() {
        //发起请求
        const {data: res} = await uni.$http.get('/api/public/v1/goods/search', this.queryObj)
        if(res.meta.status !== 200) return uni.$showMsg()
        
        //为数据赋值
        this.goodsList = res.message.goods
        this.total = res.message.total
      }
    }
  }
</script>

<style lang="scss">

</style>
