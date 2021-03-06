<!-- Registration page. Used to create new user accounts-->
<template>
  <div class="register">
    <Header
      id="titlebar"
      :title="$t('register.title')"
    />

    <b-card
      class="login-box"
      align="center"
      bg-variant="dark"
      text-variant="white"
    >
      <br>
      <h4 v-html="$t('ui.createAccount')"></h4>
      <br>
      <div>
        <b-overlay
          :show="loading"
          variant="light"
          opacity="0.6"
          blur="2px"
          rounded="sm"
        >
          <b-form @submit="onSubmit" @reset="onReset">
            <b-form-input
              id="fname"
              v-model="fname"
              :state="fnameState"
              :placeholder="$t('profile.firstName')"
              trim
              autocomplete="given-name"
            ></b-form-input>
            <b-tooltip target="fname" variant="info" triggers="hover">
              {{$t('register.malFirstName')}}
            </b-tooltip>
            <br>

            <b-form-input
              id="lname"
              v-model="lname"
              :state="lnameState"
              :placeholder="$t('profile.lastName')"
              trim
              autocomplete="family-name"
            ></b-form-input>
            <b-tooltip target="lname" variant="info" triggers="hover">
              {{$t('register.malLastName')}}
            </b-tooltip>
            <br>

            <b-form-input
              id="uname"
              v-model="uname"
              :state="unameState"
              :placeholder="$t('profile.username')"
              trim
            ></b-form-input>
            <b-tooltip target="uname" variant="info" triggers="hover">
              {{$t('register.malUsername')}}
            </b-tooltip>
            <br>

            <b-form-input
              id="email"
              v-model="email"
              :state="emailState"
              :placeholder="$t('profile.email')"
              trim
              autocomplete="email"
            ></b-form-input>
            <b-tooltip target="email" variant="info" triggers="hover">
              {{$t('register.malEmail')}}
            </b-tooltip>
            <br>

            <b-form-input
              id="passwd"
              v-model="passwd"
              :state="passwdState"
              :placeholder="$t('profile.password')"
              type="password"
              trim
              autocomplete="new-password"
            ></b-form-input>
            <b-tooltip target="passwd" variant="info" triggers="hover">
              {{$t('register.malPassword')}}
            </b-tooltip>
            <br>

            <b-button-group>
              <b-button type="submit" variant="primary" :disabled="!finalState">{{$t('ui.submit')}}</b-button>
              <b-button type="reset" variant="danger">{{$t('ui.reset')}}</b-button>
            </b-button-group>
            <br>

          </b-form>
        </b-overlay>
      </div>
      <br>

      <!-- Registered users can click here and log in -->
      <router-link to="/login">
        {{$t('ui.toLogin')}}
      </router-link>
    </b-card>

  </div>
</template>

<script>
import { axios } from '@/mixins/axios.js'
import Header from '@/components/Header.vue'

export default {
  name: 'Register',
  mixins: [axios],
  components: {
    Header
  },
  data () {
    return {
      loading: false,
      uname: '',
      email: '',
      passwd: '',
      fname: '',
      lname: ''
    }
  },
  // Computed values show whether the input strings are valid
  computed: {
    fnameState () {
      return /^[\'\-\. a-zA-ZŠŽšžŸÀ-ÖÙ-öù-ÿ]{2,20}$/.test(this.fname)
    },
    lnameState () {
      return /^[\'\-\. a-zA-ZŠŽšžŸÀ-ÖÙ-öù-ÿ]{2,20}$/.test(this.lname)
    },
    emailState () {
      return /^(?=[a-zA-Z0-9][a-zA-Z0-9@._%+-]{5,253}$)[a-zA-Z0-9._%+-]{1,64}@(?:(?=[a-zA-Z0-9-]{1,63}\.)[a-zA-Z0-9]+(?:-[a-zA-Z0-9]+)*\.){1,8}[a-zA-Z]{2,63}$/.test(this.email)
    },
    unameState () {
      return /^[a-zA-Z0-9]{5,30}$/.test(this.uname)
    },
    passwdState () {
      return /^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9]|.*[\-=._#§@$!%*?&])[A-Za-z0-9\-=._#§@$!%*?&]{8,}$/.test(this.passwd)
    },
    finalState () {
      return (this.passwdState && this.unameState && this.emailState && this.lnameState && this.fnameState)
    }
  },
  methods: {
    // Sends user input to the backend to create a new account
    onSubmit (event) {
      event.preventDefault()
      if (!this.finalState) { return }
      this.loading = true

      this.axiosPOST('api/users/register',
        {
          'userName': this.uname,
          'email': this.email,
          'password': this.passwd,
          'firstName': this.fname,
          'lastName': this.lname
        }
      )
        .then(response => {
          this.directLogin()
        })
        .catch(err => {
          this.loading = false
          switch (err.response.status) {
            case 400:
            case 500:
            default:
              this.$log.error(err)
          }
        })
    },
    // Used to reset the input fields
    onReset (event) {
      event.preventDefault()
      this.uname = ''
      this.email = ''
      this.passwd = ''
      this.fname = ''
      this.lname = ''
    },
    directLogin () {
      this.axiosPOST('api/users/login', {
        email: this.email,
        password: this.passwd
      })
        .then(response => {
          window.localStorage.setItem('mnb_atok', response.data.accessToken)
          window.localStorage.setItem('mnb_rtok', response.data.refreshToken)
          window.localStorage.setItem('mnb_uid', response.data.uid)
          window.localStorage.setItem('mnb_rid', response.data.rid)
          window.localStorage.setItem('mnb_inits', response.data.initials)
          this.$router.push({ name: 'Home' })
        })
        .catch(err => {
          switch (err.response.status) {
            case 400:
            case 403:
              break
            case 500:
            default:
              this.$log.error(err)
          }
        })
      this.loading = false
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
    height: auto;
    margin: 7vh auto auto auto;
  }
</style>
