<template>
  <view>
    <!-- 选择收货地址的盒子 -->
    <view class="address-choose-box" v-if="JSON.stringify(address) === '{}'">
      <button type="primary" size="mini" class="btnChooseAddress" @click="chooseAddress">请选择收货地址+</button>
    </view>
    
    <!-- 渲染收货信息的盒子 -->
    <view class="address-info-box" v-else @click="chooseAddress">
      <view class="row1">
        <view class="row1-left">
          <view class="username">
            收货人:{{address.userName}}
          </view>
        </view>
        <view class="row1-right">
          <view class="phone">
            电话:{{address.telNumber}}
          </view>
          <uni-icons type="arrowright" size="16"></uni-icons>
        </view>
      </view>
      <view class="row2">
        <view class="row2-left">
          收货地址:
        </view>
        <view class="row2-right">
          {{addstr}}
        </view>
      </view>
      
      <!-- 底部的边框线 -->
      <image src="/static/底部边框线.png" mode="" class="address-border"></image>
    </view>
  </view>
</template>

<script>
  import { mapState, mapMutations, mapGetters } from 'vuex'
  
  export default {
    name:"my-address",
    data() {
      return {
      };
    },
    methods: {
      ...mapMutations('m_user', ['updateAddress']),
      //选择收货地址
      async chooseAddress() {
        const [err, succ] = await uni.chooseAddress().catch(err => err)
        //用户成功的选择了收货地址
        if (succ && succ.errMsg === 'chooseAddress:ok') {
          //更新 vuex 中的收货地址
          this.updateAddress(succ)
        }
        
        //用户没有授权
        if (err && (err.errMsg === 'chooseAddress:fail auth deny' || err.errMsg === 'chooseAddress:fail authorize no response')) {
          this.reAuth() //向用户重新发起授权申请
        }
      },
      
      //调用此方法,重新发起收货地址的授权
      async reAuth() {
        //提示用户对地址进行授权
        const [err2, confirmResult] = await uni.showModal({
          content: '检测到您没有打开地址权限,是否去设置打开?',
          confirmText: '确认',
          cancelText: '取消'
        })
        
        //如果弹框异常,则直接退出
        if (err2) return
        
        //如果用户点击了 "取消" 按钮,则提示用户 "您取消了地址授权"
        if (confirmResult.cancel) return uni.$showMsg('您取消了地址授权!')
        
        //如果用户点击了 "确认" 按钮,则调用 uni.openSetting() 方法进入授权页面,让用户重新进行授权
        if (confirmResult.confirm) return uni.openSetting({
          //授权结束,需要对授权的结果做进一步判断
          success: (settingResult) => {
            //地址授权的值等于 true ,提示用户 "授权成功"
            if (settingResult.authSetting['acope.address']) return uni.$showMsg('授权成功!请选择地址')
            
            //地址授权的值等于 false ,提示用户 "您取消了地址授权"
            if (!settingResult.authSetting['acope.address']) return uni.$showMsg('您取消了地址授权')
          }
        })
      }
    },
    computed: {
      ...mapState('m_user', ['address']),
      ...mapGetters('m_user', ['addstr'])
    }
  }
</script>

<style lang="scss">
.address-border {
  display: block;
  width: 100%;
  height: 5px;
}

.address-choose-box {
  height: 90px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.address-info-box {
  font-size: 12px;
  height: 90px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 0 5px;
  
  .row1 {
    display: flex;
    justify-content: space-between;
    
    .row1-right {
      display: flex;
      justify-content: space-between;
    }
  }
  
  .row2 {
    display: flex;
    align-items: center;
    margin-top: 10px;
    
    .row2-left {
      white-space: nowrap;
    }
  }
}
</style>