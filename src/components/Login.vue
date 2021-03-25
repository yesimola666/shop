<template>
<div class="login_container">
    <div class="login_box">
        <div class="avatar_box">
            <img src="../assets/img/4.jpg" alt="" />
        </div>
        <el-form ref="loginFoemRef" class="login_form" :model="loginForm" :rules="loginFormRules">
            <!-- 用户名 -->
            <el-form-item prop="username">
                <el-input prefix-icon="iconfont icon-user" v-model="loginForm.username"></el-input>
            </el-form-item>
            <!-- 密码 -->
            <el-form-item prop="password">
                <el-input type="password" prefix-icon="iconfont icon-3702mima" v-model="loginForm.password"></el-input>
            </el-form-item>
            <el-form-item class="btns">
                <el-button type="primary" @click="login">登录</el-button>
                <el-button type="info" @click="resetLoginForm">重置</el-button>
            </el-form-item>
        </el-form>
    </div>
</div>
</template>

<script>
export default {
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        username: [{
          required: true,
          message: '请输入登录名称',
          trigger: 'blur'
        },
        {
          min: 3,
          max: 10,
          message: '长度在3到10个字符',
          trigger: 'blur'
        }
        ],
        password: [{
          required: true,
          message: '请输入登录密码',
          trigger: 'blur'
        },
        {
          min: 6,
          max: 15,
          message: '长度在6到15个字符',
          trigger: 'blur'
        }
        ]
      }
    }
  },
  methods: {
    resetLoginForm () {
      this.$refs.loginFoemRef.resetFields()
    },
    login () {
      this.$refs.loginFoemRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        const {
          data: res
        } = await this.$http.post('login', this.loginForm)
        // console.log(res)
        if (res.meta.status !== 200) return this.$message.error('登陆失败！')
        this.$message.success('登陆成功！')
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
    height: 100%;
    background-color: #2b4b6b;
    position: relative;

    .login_box {
        width: 450px;
        height: 300px;
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
        background-color: #fff;

        .avatar_box {
            width: 130px;
            height: 130px;
            background-color: #000;
            border-radius: 50%;
            position: absolute;
            left: 50%;
            transform: translate(-50%, -50%);
            box-shadow: 0 0 10px #ddd;
            padding: 10px;

            img {
                width: 100%;
                height: 100%;
                border-radius: 50%;
            }
        }

        .login_form {
            position: absolute;
            bottom: 0%;
            width: 100%;
            padding: 0 20px;
            box-sizing: border-box;

            .btns {
                display: flex;
                justify-content: flex-end;
            }
        }
    }
}
</style>
