<!-- Login page. Users can log in with their e-mail addresses or usernames -->
<template>
  <div class="login">
    <Header id="titlebar" :title="$t('login.title')" />

    <!-- Loading circle displayed while system is sending user data to the backend -->
    <b-overlay :show="loading" variant="light" opacity="0.6" blur="2px" rounded="sm">
      <b-card class="login-box" align="center" bg-variant="dark" text-variant="white">
        <br>
        <h2>{{$t('ui.welcome')}}</h2>
        <br>
        <b-form @submit="onSubmit">
          <b-form-input
            id="email"
            v-model="email"
            :state="loginState"
            :placeholder="$t('login.emailOrName')"
            trim
            autocomplete="email"
          >
          </b-form-input>
          <br>
          <b-form-input
            type="password"
            id="passwd"
            v-model="passwd"
            :state="loginState"
            :placeholder="$t('profile.password')"
            trim
            autocomplete="current-password"
          >
          </b-form-input>
          <b-tooltip
            :show.sync="tooltipState"
            target="passwd"
            variant="danger"
            placement="bottom"
            v-if="loginState === false"
            triggers="blur"
          >
            {{$t('login.invalidLogin')}}
          </b-tooltip>
          <br>
          <b-button type="submit" variant="primary">{{$t('ui.submit')}}</b-button>
        </b-form>
        <br>
        <!-- Users who don't have an account yet can register here -->
        <router-link to="/register">{{$t('ui.toSignUp')}}</router-link>
      </b-card>
    </b-overlay>
  </div>
</template>

<script>
// @ is an alias to /src
import { axios } from '@/mixins/axios.js'
import Header from '@/components/Header.vue'

export default {
  name: 'Login',
  mixins: [axios],
  components: {
    Header
  },
  data () {
    return {
      email: '',
      passwd: '',
      loginState: null,
      tooltipState: false,
      loading: false,
      loginData: {}
    }
  },
  methods: {
    // Sends user input to the backend to check if credentials are valid
    onSubmit (event) {
      event.preventDefault()
      this.loginState = null
      this.loading = true
      this.loginData.password = this.passwd;
      /.*\@.*\..*/.test(this.email)
        ? (this.loginData.email = this.email)
        : (this.loginData.userName = this.email)

      this.axiosPOST('api/users/login', this.loginData)
        .then(response => {
          this.loginState = true
          this.loading = false
          window.localStorage.setItem('mnb_atok', response.data.accessToken)
          window.localStorage.setItem('mnb_rtok', response.data.refreshToken)
          window.localStorage.setItem('mnb_uid', response.data.uid)
          window.localStorage.setItem('mnb_rid', response.data.rid)
          window.localStorage.setItem('mnb_inits', response.data.initials)
          this.$router.push({ name: 'Home' })
        })
        .catch(err => {
          this.loading = false
          switch (err.response.status) {
            case 400:
            case 403:
              this.loginState = false
              this.tooltipState = true
              break
            case 500:
            default:
              this.$log.error(err)
          }
        })
      this.loginData = {}
    }
  }
}
</script>

<style scoped>
a {
  color: var(--link);
}

.login-box {
  width: 400px;
  margin: 10vh auto auto auto;
}
</style>
