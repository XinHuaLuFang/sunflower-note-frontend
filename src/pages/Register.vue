<template>
  <section id="register-page"
           v-loading="loading"
           element-loading-text="注册中...发送激活邮件中..."
  >
    <h1>周报管理系统</h1>
    <span>注册新用户</span>
    <div style="max-width: 500px; margin: 50px auto 0; padding: 20px">
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="邮箱" prop="email">
          <el-input placeholder="邮箱" v-model="form.email"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input placeholder="密码" v-model="form.password" type="password"></el-input>
        </el-form-item>
        <el-form-item label="重复密码" prop="repeatPassword">
          <el-input placeholder="重复密码" v-model="form.repeatPassword" type="password"></el-input>
        </el-form-item>
        <el-form-item label="姓名" prop="name">
          <el-input placeholder="姓名" v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="性别" prop="sex">
          <el-select style="width: 100%" v-model="form.sex" placeholder="请选择性别">
            <el-option label="男" value="male"></el-option>
            <el-option label="女" value="female"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="小组" prop="groupId">
          <el-select style="width: 100%" v-model="form.groupId" placeholder="请选择小组">
            <el-option v-for="item in groups" :label="item.name" :value="item.id"
                       :key="item.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input placeholder="备注" v-model="form.remark"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button icon="el-icon-back" @click="gotoLogin">返回登录</el-button>
          <el-button type="primary" @click="onSubmit">确认注册</el-button>
        </el-form-item>
      </el-form>
    </div>
  </section>
</template>

<script>
  import $axios from '@/plugins/ajax'
  import Validaters from '@/common/utils/validaters';
  export default {
    data () {
      return {
        loading: false,
        form: {
          email: '',
          password: '',
          repeatPassword: '',
          name: '',
          sex: '',
          groupId: '',
          remark: '',
          mailTail: '@suninfo.com',
        },
        rules: {
          email: [
            {required: true, message: '必须', trigger: 'blur'}
          ],
          password: [{required: true, message: '必须', trigger: 'blur'}],
          repeatPassword: [
            {required: true, message: '必须', trigger: 'blur'},
            {
              validator: (rule, value, callback) => {
                if (value !== this.form.password) {
                  return callback(new Error('两次输入密码不一致！'))
                } else {
                  callback();
                }
              },
              trigger: 'blur'
            }
          ],
          name: [{required: true, message: '必须', trigger: 'blur'}],
          sex: [{required: true, message: '必须', trigger: 'blur'}],
          groupId: [{required: true, message: '必须', trigger: 'blur'}]
        },
        groups: []
      }
    },
    created () {
      $axios.get('/group/groupManage/get').then(({data}) => {
        if (data.status) {
          this.groups = data.data
        }
      })
      $axios.get('/system/getSysSetting').then(({data}) => {
        if (data.status) {
          let emailReg = '^[\\w]{1,30}(' + data.data.emailSuffix.split(';').map(item => `(${item})`).join('|') + ')$'
          console.log(emailReg)
          this.rules.email = [
            {required: true, message: '必须', trigger: 'blur'},
            {
              pattern: new RegExp(emailReg),
              message: `只能使用${data.data.emailSuffix}等邮箱后缀`,
              trigger: 'blur'
            }
          ]
        }
      })
    },
    methods: {
      onSubmit() {
        let _self = this
        this.$refs.form.validate(valid => {
          if (valid) {
            this.loading = true
            $axios.post('/user/register', this.form).then(({data}) => {
              this.loading = false
              if (data.status) {
                this.$message.success('注册成功,请前往注册邮箱查看激活邮件!')
                window.setTimeout(() => {
                  this.$router.push('/')
                }, 500)
//                this.$store.dispatch('LOGIN', {
//                  email: _self.form.email,
//                  password: _self.form.password
//                }).then(function (data) {
//                  _self.$router.push('/');
//                }).catch(function (data) {
//                  _self.$message({
//                    message: data.data,
//                    type: 'error'
//                  });
//                })
              } else {
                this.$message.error(data.data)
              }
            })
          }
        })
      },
      gotoLogin() {
        this.$router.push('/Login')
      }
    }
  }
</script>

<style>
  #register-page {
    width: 100%;
    text-align: center;
    position: absolute;
    top: 40%;
    padding-top: 80px;
    transform: translateY(-40%);
  }

  @media screen and (max-width: 768px) {
    #register-page .el-form-item__label {
      display: none;
    }

    #register-page .el-form-item__content {
      margin-left: 0 !important;
    }
  }
</style>


