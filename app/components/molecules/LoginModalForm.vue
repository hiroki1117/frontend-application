<template>
  <div>
    <h1 class="title" v-if="isSelectedEmailAuth">ログイン</h1>
    <div class="modal-body">
      <div class="email-auth" v-show="isSelectedEmailAuth" :class="{ isSelectedEmailAuth }">
        <form class="signup-form" @keypress.enter="onSubmit">
          <div class="signup-form-group" :class="{ 'error': hasUserIdOrEmailError }">
            <label class="signup-form-label">ユーザーID または メールアドレス</label>
            <input
              class="signup-form-input"
              type="text"
              placeholder="alis@example.com"
              ref="userIdOrEmail"
              @input="setUserIdOrEmail"
              @blur="showError('userIdOrEmail')"
              @focus="resetError('userIdOrEmail')">
          </div>
          <div class="signup-form-group" :class="{ 'error': hasPasswordError }">
            <label class="signup-form-label">パスワード※半角英数字8文字以上</label>
            <input
              class="signup-form-input"
              type="password"
              ref="password"
              placeholder="●●●●●●●●"
              @input="setPassword"
              @blur="showError('password')"
              @focus="resetError('password')">
            <p class="error-message" v-if="showErrorInvalidPassword">パスワードは8文字以上で入力してください</p>
          </div>
        </form>
        <div class="modal-footer">
          <p class="error-message">{{ errorMessage }}</p>

          <p class="agreement-confirmation" :class="{ isSelectedEmailAuth }">
            <nuxt-link to="/terms" target="_blank">利用規約</nuxt-link>、
            <nuxt-link to="/privacy" target="_blank">プライバシーポリシー</nuxt-link>に同意して
          </p>
          <app-button class="login-button" :disabled="invalidSubmit" @click="onSubmit">
            ログインする
          </app-button>
          <p class="for-password-forgot-user">
            パスワードを忘れた方は<span class="link" @click="forgotPassword">こちら</span>
          </p>
        </div>
      </div>
      <div class="external-provider-auth" v-show="!isSelectedEmailAuth">
        <a class="line-button" :href="lineLoginAuthorizeURL">
          LINEでログイン
        </a>
        <a class="twitter-button" :href="twitterLoginAuthorizeURL">
          twitterでログイン
        </a>
        <p
          class="for-email-login"
          @click="showEmailAuth">
          メールでログイン
        </p>
        <p class="agreement-confirmation">
          上記を押した場合、<nuxt-link to="/terms" target="_blank">利用規約</nuxt-link>・<nuxt-link to="/privacy" target="_blank">プライバシーポリシー</nuxt-link>に同意したものとみなします
        </p>
      </div>
    </div>
    <div
      class="for-signup-user"
      @click="transitToSignup"
      v-if="!isSelectedEmailAuth">
      新規登録をされる方は<span class="link-sp">こちら</span>
    </div>
    <div class="for-signup-user-sp" v-else>
      新規登録をされる方は<span class="for-signup-user-link" @click="transitToSignup">こちら</span>
    </div>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import { required, minLength } from 'vuelidate/lib/validators'
import { ADD_TOAST_MESSAGE } from 'vuex-toast'
import AppButton from '../atoms/AppButton'

export default {
  data() {
    return {
      errorMessage: '',
      lineLoginAuthorizeURL: null,
      twitterLoginAuthorizeURL: null,
      isSelectedEmailAuth: false
    }
  },
  async mounted() {
    this.lineLoginAuthorizeURL = await this.getLineLoginAuthorizeURL()
    this.twitterLoginAuthorizeURL = await this.getTwitterLoginAuthorizeURL()
  },
  components: {
    AppButton
  },
  computed: {
    showErrorInvalidPassword() {
      return this.loginModal.formError.password && !this.$v.loginModal.formData.password.minLength
    },
    invalidSubmit() {
      return this.$v.loginModal.formData.$invalid
    },
    hasUserIdOrEmailError() {
      return (
        this.loginModal.formError.userIdOrEmail && this.$v.loginModal.formData.userIdOrEmail.$error
      )
    },
    hasPasswordError() {
      return this.loginModal.formError.password && this.$v.loginModal.formData.password.$error
    },
    isShowOnlyEmailAuth() {
      return this.isShowEmailAuth && !this.isShowExternalProviderAuth
    },
    isShowOnlyExternalProviderAuth() {
      return !this.isShowEmailAuth && this.isShowExternalProviderAuth
    },
    ...mapGetters('user', ['loginModal', 'showLoginModal'])
  },
  validations: {
    loginModal: {
      formData: {
        userIdOrEmail: {
          required
        },
        password: {
          required,
          minLength: minLength(8)
        }
      }
    }
  },
  methods: {
    setUserIdOrEmail(e) {
      this.setLoginUserIdOrEmail({ userIdOrEmail: e.target.value })
    },
    setPassword(e) {
      this.setLoginPassword({ password: e.target.value })
    },
    showError(type) {
      this.$v.loginModal.formData[type].$touch()
      this.showLoginError({ type })
    },
    resetError(type) {
      this.$v.loginModal.formData[type].$reset()
      this.hideLoginError({ type })
    },
    transitToSignup() {
      this.setLoginModal({ showLoginModal: false })
      this.setSignUpModal({ showSignUpModal: true })
    },
    showEmailAuth() {
      this.isSelectedEmailAuth = true
    },
    async onSubmit() {
      if (this.invalidSubmit) return
      const { userIdOrEmail, password } = this.loginModal.formData
      try {
        await this.login({ userId: userIdOrEmail, password })
        await this.setCurrentUserInfo()
        this.setLoginModal({ showLoginModal: false })

        this.sendNotification({ text: 'ログインしました' })

        document.querySelector('html,body').style.overflow = ''
        this.$refs.userIdOrEmail.value = ''
        this.$refs.password.value = ''
        this.resetPassword()
      } catch (error) {
        let errorMessage = ''
        switch (error.code) {
          case 'NotAuthorizedException':
            errorMessage = 'ユーザーID・メールアドレスまたはパスワードが正しくありません'
            break
          default:
            errorMessage = 'エラーが発生しました。入力内容を確認してください'
            break
        }
        this.errorMessage = errorMessage
      }
    },
    ...mapActions({
      sendNotification: ADD_TOAST_MESSAGE
    }),
    ...mapActions('user', [
      'login',
      'setLoginModal',
      'setSignUpModal',
      'setLoginUserIdOrEmail',
      'setLoginPassword',
      'showLoginError',
      'hideLoginError',
      'setCurrentUserInfo',
      'resetPassword',
      'forgotPassword',
      'setSignUpAuthFlowModal',
      'setSignUpAuthFlowInputPhoneNumberModal',
      'getLineLoginAuthorizeURL',
      'getTwitterLoginAuthorizeURL'
    ])
  }
}
</script>

<style lang="scss" scoped>
.logo {
  margin: 0 auto 40px;
  display: block;
}

.modal-body {
  margin: 0 auto;
  display: flex;
}

.title {
  color: #030303;
  font-size: 20px;
  font-weight: 500;
  letter-spacing: 5px;
  margin: 0 0 20px;
  text-align: center;
  padding-top: 20px;
}

.email-auth {
  max-width: 520px;
  width: 100%;

  &.isSelectedEmailAuth {
    max-width: 100%;

    .signup-form {
      margin: 20px auto 0;
      max-width: 400px;
    }
  }
}

.signup-form {
  margin: 60px auto 0;
  width: 100%;

  &-group {
    position: relative;
  }

  &-label {
    color: #030303;
    font-size: 14px;
    line-height: 2.4;
  }

  &-input {
    appearance: none;
    box-shadow: 0 0 16px 0 rgba(192, 192, 192, 0.5);
    border: none;
    border-radius: 0;
    box-sizing: border-box;
    margin-bottom: 40px;
    padding: 12px;
    width: 100%;

    &::-webkit-input-placeholder {
      padding: 3px;
      color: #cecece;
      font-size: 14px;
      letter-spacing: 0.05em;
    }

    &:focus {
      outline: 0;
    }
  }

  .error-message {
    bottom: 20px;
    margin: 0;
    color: #f06273;
    font-size: 12px;
    position: absolute;
    width: 100%;
    text-align: right;
  }

  .error {
    .signup-form {
      &-input {
        box-shadow: 0 0 16px 0 rgba(240, 98, 115, 0.5);
      }
    }
  }
}

.modal-footer {
  width: 270px;
  margin: 20px auto 0;
  position: relative;

  .login-button {
    margin: 20px auto 0;
  }

  .error-message {
    color: #f06273;
    font-size: 12px;
    width: 100%;
  }

  .for-password-forgot-user {
    @include default-text();
    text-align: right;
  }

  .link {
    @include default-link();
  }
}

.agreement-confirmation {
  @include default-text();
  max-width: 320px;
  text-align: center;
}

.external-provider-auth {
  width: 100vw;
  display: flex;
  flex-flow: column nowrap;
  align-items: center;
  background: url('~assets/images/pc/bg/login.png') no-repeat;
  background-size: auto 370px;
  background-position-x: center;
  margin: -50px -30px 0;
}

@mixin external-provider-button {
  border-radius: 18px;
  border: none;
  box-sizing: border-box;
  color: #fff;
  cursor: pointer;
  display: block;
  font-size: 14px;
  outline: none;
  padding: 10px;
  text-align: center;
  text-decoration: none;
  width: 255px;
}

.line-button {
  margin-top: 270px;
  background: url('~assets/images/pc/common/icon_line.png') no-repeat;
  background-color: #00c300;
  background-size: 24px;
  background-position: 24px 7px;
  @include external-provider-button();
}

.twitter-button {
  margin: 20px 0 0;
  background: url('~assets/images/pc/common/icon_twitter.png') no-repeat;
  background-color: #1da1f3;
  background-size: 20px;
  background-position: 26px 10px;
  @include external-provider-button();
}

.for-email-login {
  margin: 24px 0 0;
  color: #4e4e4e;
  font-size: 14px;
  text-align: center;
  cursor: pointer;
}

.agreement-confirmation {
  @include default-text();
  text-align: center;
  margin: 20px 0 60px;

  &.isSelectedEmailAuth {
    margin: 20px 0 0;
  }
}

.for-signup-user {
  display: flex;
  color: #333;
  font-size: 12px;
  text-align: center;
  cursor: pointer;
  height: 36px;
  align-items: center;
  justify-content: center;
  margin: 0 -30px -20px;
  box-shadow: 0 0 8px 0 rgba(192, 192, 192, 0.5);
}

.for-signup-user-sp {
  font-size: 12px;
  margin: 0 auto 30px;
  text-align: right;
  width: 270px;

  .for-signup-user-link {
    @include default-link();
  }
}

@media screen and (max-width: 550px) {
  .email-auth {
    &.isSelectedEmailAuth {
      .signup-form {
        max-width: 256px;
      }
    }
  }

  .external-provider-auth {
    margin: -60px -30px 0;
  }

  .for-signup-user {
    color: #6e6e6e;
    font-size: 12px;
    margin: 10px auto 20px;
    max-width: 320px;
    text-align: right;
    box-shadow: none;
    display: block;

    .link-sp {
      color: #0086cc;
    }
  }
}

@media screen and (max-width: 320px) {
  .title {
    padding-top: 0;
  }

  .agreement-confirmation {
    margin: 10px 0 0;
    padding: 0 10px;
  }

  .external-provider-confirmation {
    margin: 10px 0;
    padding: 0 10px;
  }
}
</style>
