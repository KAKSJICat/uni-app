<template>
  <view class="login-container">
    <!-- 提示登录的图标 -->
    <uni-icons type="contact-filled" size="100" color="#afafaf"></uni-icons>
    <!-- 登录按钮 -->
    <button type="primary" class="btn-login" open-type="getUserInfo"
    @getuserinfo="getUserInfo">一键登录</button>
    <!-- 登录提示 -->
    <view class="tips-text">登录后尽享更多权益</view>
  </view>
</template>

<script>
  import { mapMutations, mapState } from 'vuex'
  
  export default {
    name:"my-login",
    data() {
      return {
        
      };
    },
    computed: {
      ...mapState('m_user', ['redirectInfo'])
    },
    methods: {
      //获取微信用户的基本信息
      getUserInfo(e) {
        //判断是否获取用户信息成功
        if (e.detail.errMsg === 'getUserInfo:fail auth deny') return uni.$showMsg('您取消了登录授权!')
        
        //获取用户信息成功
        this.updateUserInfo(e.detail.userInfo)
        
        //获取登录成功后的 Token 字符串
        this.getToken(e.detail)
      },
      
      //调用 mapMutations 辅助方法
      ...mapMutations('m_user', ['updateUserInfo', 'updateToken', 'updateRedirectInfo']),
      
      //调用登录接口,换取永久的 token
      async getToken(info) {
        //调用微信登录接口
        const [err, res] = await uni.login().catch(err => err)
        
        //判断是否 wx.login() 调用失败
        if (err || res.errMsg !== 'login:ok') return uni.$showMsg('登录失败!')
        
        //准备参数对象
        const query = {
          code: res.code,
          encryptedData: info.encryptedData,
          iv: info.iv,
          rawData: info.rawData,
          signature: info.signature
        }
        
        //换取 token
        const {data: loginResult} = await uni.$http.post('/api/public/v1/users/wxlogin', query)
        if (loginResult.meta.status !== 400) return uni.$showMsg('登录失败!')
        uni.$showMsg('登录成功!')
        
        this.navigeteBack()
      },
      
      //返回登录之前的页面
      navigateBack() {
        if (this.redirectInfo && this.redirectInfo.openType === 'switchTab') {
          uni.switchTab({
            url: this.redirectInfo.from,
            complete: () => {
              this.updateRedirectInfo(null)
            }
          })
        }
      }
    }
  }
</script>

<style lang="scss">
.login-container {
  height: 750rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #f8f8f8;
  position: relative;
  overflow: hidden;
  
  &::after {
    content: ' ';
    display: block;
    width: 100%;
    height: 40px;
    position: absolute;
    bottom: 0;
    left: 0;
    border-radius: 100%;
    transform: translateY(50%);
    background-color: white;
  }
  
  .btn-login {
    width: 90%;
    border-radius: 100px;
    margin: 15px 0;
    background-color: #c00000;
  }
  
  .tips-text {
    font-size: 12px;
    color: gray;
  }
}
</style>