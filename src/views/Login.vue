<template>
  <div class="login">
    <div class="login-con">
      <div class="image1"></div>
      <div class="image"></div>
      <div class="form-con">
        <Form ref="loginForm" :model="form" :rules="rules" @keydown.enter.native="handleSubmit">
          <FormItem prop="userName">
            <Input v-model="form.userName" placeholder="用户名" style="background:#000 !important;">
              <span slot="prepend">
                <Icon :size="16" type="ios-person"></Icon>
              </span>
            </Input>
          </FormItem>
          <FormItem prop="password">
            <Input type="password" v-model="form.password" placeholder="密码">
              <span slot="prepend">
                <Icon :size="14" type="md-lock"></Icon>
              </span>
            </Input>
          </FormItem>
          <FormItem prop="verify">
            <Input type="text" v-model="form.verify" placeholder="验证码" style="width:130px;">
              <span slot="prepend">
                <Icon :size="14" type="md-cube"></Icon>
              </span>
            </Input>
            <img
              ref="img"
              src="http://117.136.182.230/hedunwaf/get_valid_img.png/?1234"
              title="点击刷新"
              style="width: 109px;height: 32px;position: absolute;top: 0;right: 0;cursor:pointer"
              @click="refresh()"
            />
            <!-- <img
              ref="img"
              src="http://106.13.41.122:8000/hedunwaf/get_valid_img.png/?1234"
              title="点击刷新"
              style="width: 109px;height: 32px;position: absolute;top: 0;right: 0;cursor:pointer"
              @click="refresh()"
            /> -->
            <!-- <img
              src="http://106.13.41.122:8000/hedunwaf/get_valid_img.png/?1234"
              title="点击刷新"
              style="width: 109px;height: 32px;position: absolute;top: 0;right: 0;cursor:pointer"
              @click="refresh($event)"
            />-->
            <!-- <img
              src="http://117.136.182.230/hedunwaf/get_valid_img.png/?1234"
              title="点击刷新"
              style="width: 109px;height: 32px;position: absolute;top: 0;right: 0;cursor:pointer"
              @click="refresh($event)"
            />-->
          </FormItem>
          <FormItem>
            <Button @click="handleSubmit" type="primary" long>登录</Button>
          </FormItem>
        </Form>
      </div>
    </div>
  </div>
</template>
<script>
import Qs from 'qs'
export default {
  name: 'login',
  props: {
    userNameRules: {
      type: Array,
      default: () => {
        return [{ required: true, message: '账号不能为空', trigger: 'blur' }]
      }
    },
    passwordRules: {
      type: Array,
      default: () => {
        return [{ required: true, message: '密码不能为空', trigger: 'blur' }]
      }
    },
    verifyRules: {
      type: Array,
      default: () => {
        return [{ required: true, message: '验证码不能为空', trigger: 'blur' }]
      }
    }
  },
  data () {
    return {
      form: {
        userName: '',
        password: '',
        verify: ''
      },
      userToken: ''
    }
  },
  computed: {
    rules () {
      return {
        userName: this.userNameRules,
        password: this.passwordRules,
        verify: this.verifyRules
      }
    }
  },
  methods: {
    refresh (e) {
      let time = new Date().getTime()
      let src = this.$refs.img.src.substr(
        0,
        this.$refs.img.src.indexOf('?') + 1
      )
      // let src = e.target.src.substr(0, e.target.src.indexOf('?') + 1)
      // e.target.src = src + time
      this.$refs.img.src = src + time
    },
    handleSubmit () {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          let data = Qs.stringify({
            username: this.form.userName,
            password: this.form.password,
            valid_code: this.form.verify
          })
          this.axios
            .post('/login/', data)
            .then(response => {
              let resData = response.data
              this.userToken = 'Bearer' + new Date().getTime()
              this.$store.commit('changeLogin', {
                Authorization: this.userToken
              })
              if (!resData.code) {
                localStorage.setItem('user', JSON.stringify(this.form.userName))
                localStorage.setItem(
                  'role',
                  JSON.stringify(resData.data.user_type)
                )
                if (resData.data.user_type === 0) {
                  this.$router.push({ path: '/wafscreen/world' })
                } else if (resData.data.user_type === 2) {
                  this.$router.push('/normal/home')
                } else {
                  this.$router.push('/master/application')
                }
              } else {
                this.refresh()
              }
            })
            .catch(function (error) {
              console.log(error)
            })
        }
      })
    }
  },
  mounted () {
    // console.log(this.$store._mutations.changeLogin)
  }
}
</script>

<style lang="less" scoped>
.login {
  width: 100%;
  height: 100%;
  background-color: #323232;
  .image {
    width: 200px;
    height: 35px;
    margin: auto;
    background: url('../assets/img/waf_03.png') no-repeat;
  }
  .image1 {
    width: 80px;
    height: 89px;
    background: url('../assets/img/login.png') no-repeat;
    background-size: cover;
    position: absolute;
    top: -64px;
    left: 50%;
    transform: translateX(-50%);
  }
  .login-con {
    position: absolute;
    padding: 40px 40px 20px;
    left: 50%;
    top: 50%;
    border-radius: 15px;
    transform: translate(-50%, -50%);
    width: 350px;
    background-color: #484c4e;
    .login-header {
      font-size: 16px;
      font-weight: 300;
      text-align: center;
      padding: 30px 0;
    }
    .form-con {
      padding: 10px 0 0;
    }
    .login-tip {
      font-size: 10px;
      position: absolute;
      color: #ed4014;
    }
  }
}
</style>
