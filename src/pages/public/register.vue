<template>
	<view class="container">
		<view class="back-btn yticon icon-zuojiantou-up" @tap="navBack"></view>
		<view class="right-top-sign"></view>
		<!-- 设置白色背景防止软键盘把下部绝对定位元素顶上来盖住输入框等 -->
		<view class="wrapper">
			<view class="left-top-sign">REGISTER</view>
			<view class="welcome">
				账号注册！
			</view>
			<view class="input-content">
				<form @submit="toRegister">
					<view class="input-item">
						<text class="tit">手机号码</text>
						<input
							type="number"
							name="mobile"
							:value="mobile"
							placeholder="请输入手机号码"
							maxlength="11"
							@blur="blurMobileChange"
						/>
					</view>
					<view class="input-item input-item-sms-code">
						<text class="tit">验证码</text>
						<input
							type="number"
							name="code"
							placeholder="请输入验证码"
							maxlength="6"
							data-key="mobile"
						/>
						<button class="sms-code-btn" :disabled="smsCodeBtnDisabled" @tap="getSmsCode">
							<span v-if="!smsCodeBtnDisabled">获取验证码</span>
							<span v-else class="sms-code-resend">{{ `重新发送 (${codeSeconds})` }}</span>
						</button>
					</view>
					<view class="input-item">
						<text class="tit">密码</text>
						<input
							name="password"
							type="password"
							:value="password"
							placeholder="请输入密码"
							maxlength="20"
							@blur="blurPasswordChange"
						/>
					</view><view class="input-item">
						<text class="tit">确认密码</text>
						<input
							type="password"
							name="password_repetition"
							placeholder="请输入确认密码"
							maxlength="20"
							@blur="blurRePasswordChange"
						/>
					</view>
					<view class="input-item">
						<text class="tit">姓名</text>
						<input
							type="text"
							name="realname"
							placeholder="请输入您的姓名"
						/>
					</view>
					<button class="confirm-btn" formType="submit" :disabled="logining">注册</button>
				</form>
			</view>
<!--			<view class="forget-section">-->
<!--				忘记密码?-->
<!--			</view>-->
		</view>
		<view class="register-section">
			已经注册过了?
			<text @tap="navTo('/pages/public/login')">马上登录</text>
		</view>
		<view class="footer">
			注册表示同意
			<text class="protocol" @tap="navTo(`/pages/about/detail?field=protocol_register&title=注册协议`)">RangeFrame使用协议 / 注册协议</text>
		</view>
	</view>
</template>

<script>
	import {
        mapMutations
    } from 'vuex';
	import {registerByPass, smsCode} from "@/api/login";
	const graceChecker = require("@/common/graceChecker.js");
	export default{
		data(){
			return {
				mobile: '',
				password: '',
				logining: false,
				smsCodeBtnDisabled: false,
				codeSeconds: 60
			}
		},
		onLoad(){

		},
		methods: {
			...mapMutations(['login']),
			blurMobileChange(e) {
				this.mobile = e.detail.value
			},
			blurPasswordChange(e) {
				this.password = e.detail.value
			},
			blurRePasswordChange(e) {
				if (this.password !== e.detail.value) {
			    this.$api.msg('两次输入的密码不一致');
				}
			},
			navBack(){
				uni.navigateBack();
			},
			navTo(url){
				// this.$api.msg('去注册');
				uni.navigateTo({
					url
				})
			},
			/**
			 *@des 获取手机验证码
			 *@author stav stavyan@qq.com
			 *@blog https://stavtop.club
			 *@date 2019/11/15 11:10:28
			 *@param mobile
			 */
			getSmsCode () {
				if (!this.checkPhoneIsValid(this.mobile)) return;
				this.$post(smsCode, {
					mobile: this.mobile,
					usage: 'register'
				}).then(r=>{
			    this.$api.msg(`验证码发送成功, 验证码是${r.data}`);
					this.smsCodeBtnDisabled = true;
					let time = 59;
					let timer = setInterval(() => {
						if(time === 0) {
							clearInterval(timer);
							this.smsCodeBtnDisabled = false;
						} else {
							this.codeSeconds = time;
							this.smsCodeBtnDisabled = true;
							time--
						 }
					},1000)
				}).catch(err => {
					console.log(err)
				})
			},
			/**
			 *@des 验证手机号是否有效
			 *@author stav stavyan@qq.com
			 *@blog https://stavtop.club
			 *@date 2019/11/15 11:54:25
			 *@param mobile 手机号
			 */
			checkPhoneIsValid (mobile) {
				if (!mobile.length < 0) {
			    this.$api.msg(`请输入11位的手机号`);
					return false;
				}
				const reg = /^1[0-9]{10,10}$/;
				if (!reg.test(mobile)) {
			    this.$api.msg(`请输入正确的手机号`);
					return false;
				} else {
					return true;
				}
			},
			async toRegister(e){
				const formData = e.detail.value;
				if (formData.password !== formData.password_repetition) {
			    this.$api.msg(`两次输入的密码不一致`);
					return;
				}
				const rule = [
					{name:"mobile", checkType : "phoneno", checkRule:"11,11",  errorMsg:"请输入正确的手机号"},
					{name:"code", checkType : "notnull", checkRule:"",  errorMsg:"请输入验证码"},
					{name:"password", checkType : "string", checkRule:"6,20",  errorMsg:"密码长度为6-20位"},
					{name:"password_repetition", checkType : "notnull", checkRule:"",  errorMsg:"请输入确认密码"},
					{name:"realname", checkType : "string", checkRule:"2,6",  errorMsg:"姓名应为2-6个字符"}
				];
				const checkRes = graceChecker.check(formData, rule);
				if(!checkRes){
			    this.$api.msg(graceChecker.error);
					return;
				}
				let params = {}
				/*  #ifdef  APP-PLUS  */
				params.group = 'tinyShopApp'
				/*  #endif  */
				/*  #ifdef H5  */
				params.group = 'tinyShopH5'
				/*  #endif  */
				/*  #ifdef  MP-WEIXIN  */
				params.group = 'tinyShopWechatMq'
				/*  #endif  */
				/*  #ifdef  MP-QQ  */
				params.group = 'tinyShopQqMq'
				/*  #endif  */
				this.$post(registerByPass, {
					...params,
					...formData
				}).then(r=>{
			    this.$api.msg(`恭喜您注册成功`);
					uni.navigateTo({
						url: '/pages/public/login'
					})
				}).catch(err => {
					console.log(err)
				})
			}
		},

	}
</script>

<style lang='scss' scoped>
.container{
  padding-top: 60px;
  position:relative;
  width: 100vw;
  /*height: 100vh;*/
  overflow: hidden;
  background: #fff;
  .wrapper{
    position:relative;
  	width: 100vw;
    z-index: 90;
    background: #fff;
    padding-bottom: 40upx;
    .welcome{
      position:relative;
      left: 50upx;
      top: -90upx;
      font-size: 46upx;
      color: #555;
      text-shadow: 1px 0px 1px rgba(0,0,0,.3);
    }
    .input-content{
      padding: 0 60upx;
    }
    .input-item{
      display:flex;
      flex-direction: column;
      align-items:flex-start;
      justify-content: center;
      padding: 0 30upx;
      background:$page-color-light;
      height: 120upx;
      border-radius: 4px;
      margin-bottom: 50upx;
      &:last-child{
        margin-bottom: 0;
      }
      .tit{
        height: 50upx;
        line-height: 56upx;
        font-size: $font-sm+2upx;
        color: $font-color-base;
      }
      input{
        height: 60upx;
        font-size: $font-base + 2upx;
        color: $font-color-dark;
        width: 100%;
      }
    }
    .input-item-sms-code {
      position: relative;
      .sms-code-btn {
        position: absolute;
        right: 20upx;
				color: #111;
        bottom: 20upx;
        font-size: 28upx;
      }
      .sms-code-resend {
        color: #999;
      }
    }
    .confirm-btn{
      width: 630upx;
      height: 76upx;
      line-height: 76upx;
      border-radius: 50px;
      margin-top: 70upx;
      background: $uni-color-primary;
      color: #fff;
      font-size: $font-lg;
      &:after{
        border-radius: 100px;
      }
    }
    .forget-section{
      font-size: $font-sm+2upx;
      color: $font-color-spec;
      text-align: center;
      margin-top: 40upx;
    }
  }
  .back-btn{
  position:absolute;
  left: 40upx;
  z-index: 9999;
  padding-top: var(--status-bar-height);
  top: 40upx;
  font-size: 40upx;
  color: $font-color-dark;
}
  .left-top-sign{
    font-size: 120upx;
    color: $page-color-base;
    position:relative;
    left: -16upx;
  }
  .right-top-sign{
    position:absolute;
    top: 80upx;
    right: -30upx;
    z-index: 95;
    &:before, &:after{
      display:block;
      content:"";
      width: 400upx;
      height: 80upx;
      background: #b4f3e2;
    }
    &:before{
      transform: rotate(50deg);
      border-radius: 0 50px 0 0;
    }
    &:after{
      position: absolute;
      right: -198upx;
      top: 0;
      transform: rotate(-50deg);
      border-radius: 50px 0 0 0;
      /* background: pink; */
    }
  }
  .register-section{
    margin: 30upx 0 50upx;
    width: 100%;
    font-size: $font-sm+2upx;
    color: $font-color-base;
    text-align: center;
    text{
      color: $font-color-spec;
      margin-left: 10upx;
    }
  }
}
.footer {
			width: 100%;
			text-align: center;
			margin: 20upx 0 20upx;
			font-size: $font-sm + 2upx;
			.protocol {
				color: $base-color;
				margin: 0 10upx;
			}
	}
</style>
