<template>
  <view>
    <!-- 状态栏 -->
    <uni-status-bar bgcolor="#FFE933"></uni-status-bar>
    <!-- 关闭按钮 -->
    <view class="icon iconfont icon-guanbi" @tap="back"></view>
    <!-- 引入背景图 -->
    <!-- <image class="loginhead" src="../../static/common/loginhead.png" mode="widthFix" lazy-load></image> -->
    <view class="loginhead" :style="{ height: setHeight * 0.3 + 'px' }">
      <view class="txt">
        <text>Welcome</text>
        <text>Back!</text>
      </view>
      <image src="../../static/png/1.png" class="illustration"></image>
    </view>

    <!-- 输入框+按钮 -->
    <view class="body-card" :style="{ height: setHeight * 0.72 + 'px' }">
      <view class="body">
        <!-- 标题 -->
        <template>
          <view class="title"> 登 录 </view>
        </template>
        <!-- 登录框 -->
        <template v-if="!status">
          <view class="login-box">
            <view class="inputShow">
              
              <input type="text" placeholder="请输入账号" v-model="username" />
              <view class="icon">
                <image
                  src="../../static/png/clear.png"
                  @tap="clear"
                  v-if="username.length"
                  class="icon-clear"
                ></image>
              </view>
            </view>
            <view class="inputShow">
              <input
                :type="showPass ? 'text' : 'password'"
                placeholder="请输入密码"
                v-model="password"
                style="border: none; outline: none"
              />
              <image
                :src="showPass ? src1 : src2"
                @tap="showEye"
                class="icon"
              ></image>
            </view>
          </view>
        </template>
        <!-- 手机验证码登录 -->
        <template v-else>
         
          <view class="login-input-box">
            <view class="phone u-f-ajc" v-if="phone.length">手机号</view>
            <input
              type="text"
              v-model="phone"
              class="phone-input"
              placeholder="手机号"
            />
          </view>
          <view class="verify">
            <input
              type="text"
              v-model="checknum"
              class="forget-input"
              placeholder="请输入验证码"
            />
            <view
              class="forget u-f-ajc login-font-color yanzhengma"
              @tap="getCheckNum"
            >
              <view class="u-f-ajc">{{
                !codetime ? '获取验证码' : codetime + ' s'
              }}</view>
            </view>
          </view>
        </template>
        <!-- 其他 -->
        <template>
          <view class="other">
            <view @tap="changeStatus">{{
              status ? '账号密码登录' : '验证码登录'
            }}</view>
            <view>忘记密码？</view>
          </view>
        </template>
        <!-- 登录按钮 -->
        <template>
          <button
            class="user-set-btn"
            :loading="loading"
            :class="{ 'user-set-btn-disable': disabled }"
            type="primary"
            @tap="submit"
            :disabled="disabled"
          >
            登录
          </button>
        </template>
        <!-- 其他方式 -->
        <template> 其他方式 </template>
        <!-- 图标 -->
        <template>
          <view class="other-login">
            <image
              src="../../static/png/google.png"
              class="other-login-item"
            ></image>
            <image
              src="../../static/png/facebook.png"
              class="other-login-item"
            ></image>
          </view>
        </template>
        <!-- 说明 -->
        <template>
          <view class="regist">
            <view class="new-user"> New Uer? </view>
            <view class="create-account"> Create Account </view>
          </view>
        </template>
      </view>
    </view>
  </view>
</template>

<script>
import uniStatusBar from '../../components/uni-status-bar/uni-status-bar.vue'
import otherLogin from '../../components/home/other-login.vue'
export default {
  components: {
    uniStatusBar,
    otherLogin,
  },
  data() {
    return {
      status: false, //false代表账号密码登录，true代表手机验证码登录
      disabled: true,
      loading: false,
      username: '',
      password: '',
      phone: '',
      checknum: '',
      codetime: 0,
      setWidth: 0,
      setHeight: 0,
      src1: '../../static/png/open_eye.png',
      src2: '../../static/png/close_eye.png',
      showPass: false,
    }
  },
  watch: {
    username(val) {
      this.OnBtnChange()
    },
    password(val) {
      this.OnBtnChange()
    },
    phone(val) {
      this.OnBtnChange()
    },
    checknum(val) {
      this.OnBtnChange()
    },
  },
  mounted() {
    //     this.$refs.setPlan.open()
    //     // 注意，这里要用个变量存this，不然进到getSystemInfo后this指向会变化，找不到data变量
    //     var _this = this
    //     uni.getSystemInfo({
    //       success: function (res) {
    //         _this.setWidth = res.windowWidth
    //         _this.setHeight = res.windowHeight
    // 		console.log(_this.setWidth)
    //       }
    //     })
  },
  onReady() {
    // 计算屏幕剩余高度  填补剩余高度
    let _this = this
    uni.getSystemInfo({
      success(res) {
        _this.setHeight = res.windowHeight
        console.log(_this.setHeight)
      },
    })
    console.log('w' + this.scrollviewHigh)
  },
  methods: {
    showEye() {
      this.showPass = !this.showPass
    },
    clear() {
      this.username = ''
    },

    // 验证手机号码
    isPhone() {
      let mPattern = /^1[34578]\d{9}$/
      return mPattern.test(this.phone)
    },
    // 获取验证码
    async getCheckNum() {
      if (this.codetime > 0) return
      // 验证手机号合法性
      if (!this.isPhone()) {
        return uni.showToast({ title: '请输入正确的手机号码', icon: 'none' })
      }
      // 请求服务器，发送验证码
      let [err, res] = await this.$http.post('/user/sendcode', {
        phone: this.phone,
      })
      // 请求失败处理
      this.$http.errorCheck(err, res)
      if (res.data.errorCode === 30001) return
      // 发送成功，开启倒计时
      this.codetime = 60
      let timer = setInterval(() => {
        this.codetime--
        if (this.codetime < 1) {
          clearInterval(timer)
          this.codetime = 0
        }
      }, 1000)
    },
    // 初始化表单
    initInput() {
      this.username = ''
      this.password = ''
      this.phone = ''
      this.checknum = ''
    },
    // 改变按钮状态
    OnBtnChange() {
      if ((this.username && this.password) || (this.phone && this.checknum)) {
        this.disabled = false
        return
      }
      this.disabled = true
    },
    // 切换登录状态
    changeStatus() {
      this.initInput()
      this.status = !this.status
    },
    // 返回上一步
    back() {
      uni.navigateBack({ delta: 1 })
    },
    // 提交登录
    submit() {
      // 账号密码登录
      if (!this.status) {
        return this.User.login({
          url: '/user/login',
          data: {
            username: this.username,
            password: this.password,
          },
        })
      }
      // 验证码登录
      // 验证手机号合法性
      if (!this.isPhone()) {
        return uni.showToast({ title: '请输入正确的手机号码', icon: 'none' })
      }
      return this.User.login({
        url: '/user/phonelogin',
        data: {
          phone: this.phone,
          code: this.checknum,
        },
      })
    },
  },
}
</script>

<style>
.login-font-color {
  color: #bbbbbb;
}
.login-padding {
  padding: 20upx 0;
}
.icon-guanbi {
  position: fixed;
  top: 60upx;
  left: 30upx;
  font-size: 40upx;
  font-weight: bold;
  color: #332f0a;
  z-index: 100;
}
.loginhead {
  width: 100%;

  /* border: 1px red solid; */
  display: flex;
  justify-content: space-around;
  align-items: center;
  background-color: rgb(22, 135, 167);
}
.loginhead .txt {
  /* width: 50%; */
  color: white;
  font-size: large;
  font-weight: 800;
  /* border: 1px red solid; */
}
.loginhead .illustration {
  width: 200px;
  height: 200px;
  /* border: 1px red solid; */
}

.other-login-title {
  position: relative;
}

.login-input-box {
  width: 65%;
  display: flex;
  flex-direction: column;
  /* align-items: center; */
  justify-content: left;
  /* border: 1px red solid; */
}
.login-input-box .phone {
  
  display: flex;
  justify-content: left;
  align-items: center;
  /* border: 1px red solid; */
}
.verify {
  width: 65%;
  display: flex;
  /* border: 1px red solid; */
}
.yanzhengma view {
  background: #eeeeee;
  border-radius: 10upx;
  font-size: 25upx;
  width: 150upx;
  padding: 10upx 0;
}

.login-input {
  padding-right: 150upx;
}
.body-card {
  margin-top: -10%;
  background-color: white;
  padding-top: 40upx;
  /* border: 1px red solid; */
  border-radius: 40upx 40upx 0 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}
.title {
  /* border: 1px red solid; */
  display: flex;
  justify-content: center;
  align-content: center;
  font-size: x-large;
  font-weight: bold;
  width: 100%;
}
.login-box {
  /* border: 1px red solid; */
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 80%;
}
.inputShow {
  margin-top: 60upx;
  width: 100%;
  /* border: 1px red solid; */
  display: flex;
  justify-content: space-between;
  border-bottom: 1px gray solid;
}
.inputShow input {
}
.icon {
  width: 20px;
  height: 20px;
  margin-right: 10px;
}
.icon-clear {
  width: 20px;
  height: 20px;
}
.other {
  margin-top: 50upx;
  /* border: 1px red solid; */
  display: flex;
  font-weight: bolder;
  width: 80%;
  color: rgb(105, 91, 135);
  justify-content: space-between;
}
.other-login {
  /* border: 1px red solid; */
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  margin-bottom: 12upx;
}
.other-login-item {
  width: 80upx;
  height: 80upx;
  margin: 0 16upx;
  /* padding: 2upx;
  border: 2upx gray solid;
  border-radius: 50%; */
}
/* 公共按钮 */
.user-set-btn {
  width: 70%;
  margin: 25upx 0;
  background: rgb(22, 135, 167) !important;
  /* border: 2upx!important; */
  /* border: 1px red solid; */
  border-radius: 10upx;
  font-weight: 600;
  color: #333333 !important;
}
.user-set-btn-disable {
  font-weight: 600;
  background: rgb(22, 135, 167) !important;
  /* border: 1upx solid #EEEEEE!important; */
  color: white !important;
  border-radius: 10upx;
}

.body {
  padding: 0 20upx !important;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  height: 100%;
  width: 100%;
  /* border: 1px red solid; */
}
.common-input {
  font-size: 30upx;
  border-bottom: 1upx solid #f4f4f4;
}
.regist {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 60%;
  /* border: 1px red solid; */
  font-weight: bold;
  margin-bottom: 20px;
}
.new-user {
  color: rgb(135, 124, 157);
  /* border: 1px red solid; */
}
.create-account {
  color: rgb(25, 128, 158);
}
</style>
