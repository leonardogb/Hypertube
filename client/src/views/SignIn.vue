<template lang="pug">
//- .content(:style="{ backgroundImage: 'url(' + getBackgrounds + ')'}")
.content
  img.fondo(:src="getBackgrounds")
  .formulario
    h1.titulo Sign In
    el-form(@submit.native.prevent='submitForm("ruleForm2")', :model='ruleForm2', status-icon='', :rules='rules2', ref='ruleForm2')
      el-form-item(label='Login', prop='login')
        el-input(type='text', v-model='ruleForm2.login', autocomplete='off')
      el-form-item(label='Password', prop='password')
        el-input(type='password', v-model='ruleForm2.password', autocomplete='off')
      .auth
          a(href=`${process.env.VUE_APP_URI}/auth/connect`) 
            img(src='github.png')
          a(href=`${process.env.VUE_APP_URI}/auth42/connect`) 
            img(src='fortyTwo.png').logo
          a(href=`${process.env.VUE_APP_URI}/authGoogle/connect`) 
            img(src='google.png').logo
      .signBtn
        a.lien(@click="showModal") Forgot password ?
        router-link.lien(to="signup") Create account
      .submitBtn
        el-input.button-custom(type='submit' value="Sign in")
      .el-alert(v-if="error" type="error" title="error alert" class="error") {{ error }}
  resetPassword(v-show='isModalVisible', @close='closeModal')
</template>

<style scoped>

.content {
  background-position: center center;
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-size: cover;
  display: flex;
  justify-content: center;
  align-items: center;
  height: calc(100vh - 51px);
  opacity: .8;
  overflow: hidden;
}

.formulario {
  display: flex;
  width: 450px;
  flex-direction: column;
  padding: 60px;
  background-color: #000000ba;
  box-shadow: 0 10px 30px rgba(0,0,0,0.19), 0 10px 10px rgba(0,0,0,0.23);
}

.auth {
  display: flex;
  justify-content: space-around;
}

.auth a:hover {

  transition: all .3s;
}

.signBtn {
  display: flex;
  flex-direction: column;
}

.lien {
  color: #737373;
  text-decoration: none;
  cursor: pointer;
}

.lien:hover {
  color: white;
}

.titulo {
  color: whitesmoke;
}

.fondo {
  position: absolute;
  z-index: -1;
  object-fit: cover;
  width: 100%;
  height: calc(100vh - 51px);
}

input[type=submit]:hover {
  background-color: #e50914;
  color: white;
}

.linksAuth {
  display: flex;
  justify-content: space-around;
}

.logo {
  width: 80px;
}

</style>

<script>
import resetPassword from '@/components/resetPassword.vue';
import { GetBackgrounds, SignIn } from '@/constants/query.gql';
import store from '@/store.js';
import { mapGetters } from 'vuex';
import axios from 'axios';

export default {
  store,
  components: {
    resetPassword,
  },
  computed: {
    ...mapGetters([
        'messageError',
        'messageSuccess',
      ])
  },
  data() {
    let checkLogin = (rule, value, callback) => {
      if (!value) {
          return callback(new Error('Please fill the login input.'));
        }
        else {
          callback();
        }
    };
    let checkPassword = (rule, value, callback) => {
      if (!value) {
          return callback(new Error('Please fill the password input.'));
        }
        else {
          callback();
        }
    };
    return {
      isModalVisible: false,
      error: false,
      getBackgrounds: '',
      ruleForm2: {
        login: '',
        password: '',
      },
      rules2: {
        login: [
          { validator: checkLogin, trigger: 'blur' }
        ],
        password: [
          { validator: checkPassword, trigger: 'blur' }
        ]
      },
    }
  },
  apollo: {
    getBackgrounds: {
      query: GetBackgrounds,
      update: result => result.getBackgrounds
    }
  },
  mounted() {
    if (this.messageError !== '') {
      this.displayMessageError();
      store.commit('FILL_ERROR', '');
    }
    if (this.messageSuccess !== '') {
      this.displayMessageSuccess();
      store.commit('FILL_SUCCESS', '');
    }
    if (this.$route.query.error)
      this.error = this.$route.query.error;
    else
      this.error = '';
  },
  methods: {
    displayMessageError() {
      this.$message({
        showClose: true,
        message: this.messageError,
        type: 'error'
      });
    },
    displayMessageSuccess() {
      this.$message({
        showClose: true,
        message: this.messageSuccess,
        type: 'success'
      });
    },
    showModal() {
      this.isModalVisible = true;
    },
    closeModal() {
      if (this.messageError !== '') {
        this.displayMessageError();
        store.commit('FILL_ERROR', '');
      }
      if (this.messageSuccess !== '') {
        this.displayMessageSuccess();
        store.commit('FILL_SUCCESS', '');
      }
      this.isModalVisible = false;
    },
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.$apollo.query({
            query: SignIn,
            variables: {
              login: this.ruleForm2.login,
              password: this.ruleForm2.password,
            }
          }).then(async ({data}) => {
            await this.saveToken(data.signIn);
            this.$router.push('/');
            this.submitted = true;
            this.error = false;
            this.$store.dispatch('new_token_created', true);
          }).catch((error) => {
            this.ErrorHandler(error);
          })
        } else {
          return false;
        }
      });
    },
    saveToken(token) {
      localStorage.setItem('user-token', token);
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    }
    
  }
}
	
</script>
