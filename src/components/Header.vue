<template>
  <div class="header" :class="frosted && 'frosted'">
    <div class="link-container">
      <router-link :to="{ name: 'Landing' }" class="link">
        <img src="../assets/logo.png">
      </router-link>
      <router-link :to="{ name: 'Landing' }" class="link text-link">
        ArcLight
      </router-link>
      <router-link :to="{ name: 'Songs' }" class="link text-link">
        Music
      </router-link>
      <router-link :to="{ name: 'About' }" class="link text-link">
        About
      </router-link>
    </div>
    <div class="mobile-nav mobile-search-bar">
      <v-btn icon color="white" @click="mobileDialog = true">
        <v-icon>mdi-magnify</v-icon>
      </v-btn>
    </div>
    <Search class="search-bar"/>
    <v-btn v-if="!isLoggedIn" depressed large color="#E56D9B" class="sign" @click="show = true" :outlined="loginBtnLoading" :loading="loginBtnLoading">Login</v-btn>
    <v-btn v-if="isLoggedIn" class="mobile-nav mobile-upload" fab dark x-small color="#E56D9B" @click="uploadMusic">
      <v-icon dark>mdi-upload</v-icon>
    </v-btn>
    <v-btn
      v-if="isLoggedIn"
      class="upload"
      depressed
      large
      color="#E56D9B"
      :outlined="true"
      @click="uploadMusic"
    >
      <v-avatar :size="24">
        <v-icon :size="24" light style="color: #E56D9B;">mdi-upload</v-icon>
      </v-avatar>
      Upload Music
    </v-btn>

    <v-menu v-if="isLoggedIn" offset-y dark class="user-menu">
      <template v-slot:activator="{ on, attrs }">
        <v-btn
          class="mobile-nav mobile-user"
          dark
          fab
          x-small
          color="#FFF"
          v-bind="attrs"
          v-on="on"
        >
          <miniAvatar
            :size="30"
            v-if="userAvatar"
            :src="userAvatar"
          />
        </v-btn>
        <v-btn
          depressed
          class="user"
          large
          color="#E56D9B"
          v-bind="attrs"
          v-on="on"
        >
          <v-avatar size="24" style="margin-right: 0.3rem;">
            <v-icon :size="24" v-if="!userAvatar" dark>mdi-account-circle</v-icon>
            <img
              v-if="userAvatar"
              :src="userAvatar"
              :alt="username"
            >
          </v-avatar>
          <p class="username">{{ username }}</p>
        </v-btn>
      </template>
      <v-list>
        <v-list-item
          v-for="(item, index) in menuItems"
          :key="index"
          @click="goto(item)"
        >
          <v-list-item-title v-if="!item.type">{{ item.title }}</v-list-item-title>
          <v-list-item-title v-if="item.type" :class="item.type">{{ item.title }}</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-menu>

    <v-btn
      v-if="!isLoggedIn"
      fab
      dark
      x-small
      color="#E56D9B"
      class="mobile-nav mobile-sign"
      @click="show = true"
      :outlined="loginBtnLoading"
      :loading="loginBtnLoading"
    >
      <v-icon dark>mdi-login</v-icon>
    </v-btn>

    <v-dialog
      v-model="show"
      max-width="340"
    >
      <v-card>
        <v-card-title class="headline">Upload your key</v-card-title>
        <v-file-input
          v-model="file"
          accept="application/json"
          label="Upload"
          style="width: 90%; margin-left: 10px;"
          placeholder="Insert your wallet key"
        >
        </v-file-input>
        <v-checkbox
          color="#E56D9B"
          v-model="writeCookie"
          label="Save Key for this Session for 7 days"
          style="padding:0 20px;"
        >
        </v-checkbox>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            depressed
            color="#ff92bc"
            @click="submit"
            class="upload-btn"
          >
            Upload
          </v-btn>
          <v-btn
            text
            @click="show = false"
          >
            Close
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="mobileDialog" fullscreen hide-overlay transition="dialog-bottom-transition">
      <v-card dark>
        <v-toolbar dark color="#E56D9B">
          <v-btn icon dark @click="mobileDialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
          <v-toolbar-title style="margin-right: 10px;">Search</v-toolbar-title>
        </v-toolbar>
        <v-list three-line subheader>
          <Search @should-close="closeMobileDialog" class="mobile-search" />
        </v-list>
      </v-card>
    </v-dialog>
    <v-snackbar
      v-model="snackbar"
      color="#00C853"
      timeout="3000"
      top="top"
    >
      File Read Successful

      <template v-slot:action="{ attrs }">
        <v-btn
          dark
          text
          v-bind="attrs"
          @click="snackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
    <v-snackbar
      v-model="failSnackbar"
      color="#E53935"
      timeout="3000"
      top="top"
    >
      {{ failMessage }}

      <template v-slot:action="{ attrs }">
        <v-btn
          dark
          text
          v-bind="attrs"
          @click="snackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>

<script>

import { mapActions, mapState } from 'vuex'

import Search from './Search.vue'
import miniAvatar from '@/components/User/MiniAvatar'

import { clearCookie, getCookie, setCookie } from '../util/cookie'

export default {
  components: {
    Search,
    miniAvatar
  },
  data () {
    return {
      show: false,
      loginBtnLoading: false,
      writeCookie: false,
      file: null,
      keyFile: '',
      fileName: '',
      fileContent: '',
      fileRaw: '',
      needUpload: false,
      snackbar: false,
      failSnackbar: false,
      failMessage: '',
      menuItems: [
        { title: 'My Profile', path: '/user/' },
        { title: 'My Library', path: '/library' },
        { title: 'Sign Out', type: 'danger' }
      ],
      frosted: false,
      mobileDialog: false,
      showPosition: 1
    }
  },
  computed: {
    ...mapState(['isLoggedIn', 'username', 'userAvatar', 'wallet', 'userNoBalanceFailure', 'userAccountFailure'])
  },
  watch: {
    wallet (val) {
      this.menuItems[0].path = '/user/' + val
    },
    userNoBalanceFailure (val) {
      if (val) {
        this.loginBtnLoading = false
        this.failSnackbar = true
        this.failMessage = 'Account has no balance, try another one'
      }
    },
    userAccountFailure (val) {
      if (val) {
        this.failSnackbar = true
        this.failMessage = 'Account has errored transactions, check your balance'
      }
    }
  },
  mounted () {
    this.$nextTick(() => {
      this.scrollShow()
      window.addEventListener('scroll', this.scrollShow)
    })

    this.loginBtnLoading = true
    const c = getCookie('arclight_userkey')
    if (c) {
      this.setKey({
        file: 'keyFile',
        raw: c,
        name: 'key',
        content: JSON.parse(c)
      })
      this.loginBtnLoading = false
    }
    this.loginBtnLoading = false
    if (this.wallet) {
      this.menuItems[0].path = '/user/' + this.wallet
    }
  },
  methods: {
    ...mapActions(['setKey', 'logout']),
    scrollShow () {
      const currentTop = document.body.scrollTop || document.documentElement.scrollTop
      if (currentTop > this.showPosition) this.frosted = true
      else this.frosted = false
    },
    submit () {
      this.loginBtnLoading = true
      this.keyFile = this.file
      this.fileName = this.keyFile.name
      const reader = new FileReader()
      reader.readAsText(this.keyFile)
      reader.onload = async (e) => {
        try {
          this.fileContent = JSON.parse(e.target.result)
          this.fileRaw = JSON.stringify(this.fileContent)
          const data = {
            file: this.file,
            raw: this.fileRaw,
            name: this.fileName,
            content: this.fileContent
          }
          await this.setKey(data)
          this.needUpload = false
          this.snackbar = true
          this.show = false
          if (this.writeCookie) {
            clearCookie('arclight_userkey')
            setCookie('arclight_userkey', this.fileRaw, 7)
          }
        } catch (err) {
          this.failSnackbar = true
          this.failMessage = 'File Read Failed, Try again'
        }
      }
      this.menuItems[0].path = '/user/' + this.wallet
    },
    goto (item) {
      if (item.type) {
        this.loginBtnLoading = false
        clearCookie('arclight_userkey')
        this.logout()
      } else {
        this.$router.push({ path: item.path })
      }
    },
    uploadMusic () {
      this.$router.push({ name: 'Upload' })
    },
    closeMobileDialog (status) {
      this.mobileDialog = status
    }
  }
}
</script>

<style lang="less" scoped>

.mobile-nav {
  display: none;
}

.header {
  padding: 20px 0 0;
  display: flex;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 10;
  overflow: hidden;
  transition: all 0.3s;
  backdrop-filter: blur(0px);
  &.frosted {
    padding: 5px 0;
    background: #adadad4d;
    box-shadow: 3px 3px 6px 3px rgba(0, 0, 0, .3);
    backdrop-filter: blur(20px);
  }
}

.link-container {
  margin-left: 2rem;
  width: 100%;
  display: flex;
  align-items: center;
}

.link {
  margin-left: .75rem;
  font-size: 1.2rem;
  color: white;
  img {
    height: 2.2rem;
  }
}

.text-link {
  margin-bottom: 5px;
}

.text-link:hover {
  animation-name: colorChange;
  animation-duration: 0.25s;
  animation-timing-function: ease-in-out;
  color: #ff92bc;
}

@keyframes colorChange {
  0% {
    color: white;
  }
  100% {
    color: #ff92bc;
  }
}

.search-bar {
  width: 50%;
}

.mobile-search {
  margin-top: 16px;
}

.sign {
  margin-left: 1rem;
  margin-right: 2rem;
  /deep/ .v-btn__content {
    color: white;
  }
}

.user {
  margin-left: 1rem;
  margin-right: 2rem;
  /deep/ .v-btn__content {
    color: white;
    max-width: 200px;
    .username {
      margin: 0;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
    }
  }
}

.danger {
  color: #E53935 !important;
}

.upload-btn {
  /deep/ .v-btn__content {
    color: white;
  }
}

/deep/ .v-input--selection-controls {
  margin-top: 0px;
}

/deep/ .v-card.v-sheet.theme--light {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.upload {
  margin-left: 1rem;
  /deep/ i {
    color: white;
  }
}

/deep/ a {
  color: white;
  text-decoration: none;
}

.autocomplete {
  /deep/ .v-label.theme--light{
    color: white;
  }

  /deep/ .v-icon.mdi-menu-down.theme--light {
    color: white;
    &.primary--text {
      color: #E56D9B !important;
      caret-color: #E56D9B !important;
    }
  }
}

@media screen and (max-width: 1200px) {
  .link {
    font-size: 20px;
    img {
      height: 2rem;
    }
  }
  .upload {
    padding: 0 10px !important;
    height: 38px !important;
    font-size: 12px;
    /deep/ i {
      font-size: 18px !important;
    }
  }
  .user {
    padding: 0 10px !important;
    height: 38px !important;
    font-size: 12px;
  }
  .search-bar {
    display: none;
  }

  .mobile-search-bar {
    display: block;
    margin-right: 10px;
  }
}

@media screen and (max-width: 768px) {
  .link {
    font-size: 16px;
  }
  .upload {
    display: none;
  }
  .mobile-upload {
    display: block;
    margin-right: 16px;
  }
  .user {
    display: none;
  }
  .mobile-user {
    display: block;
    margin-right: .75rem;
  }
  .sign {
    display: none;
  }
  .mobile-sign {
    display: block;
    margin-right: .75rem;
  }
}

@media screen and (max-width: 640px) {
  .link-container {
    margin-left: 0.5rem;
  }
}

@media screen and (max-width: 480px) {
  .link {
    font-size: 15px;
    img {
      height: 1.8rem;
    }
  }
  .link-container {
    margin-left: 8px;
    margin-top: 5px;
  }
}
</style>

<style lang="less">
/deep/ .theme--light .v-list {
  background-color: #333;
}

@media screen and (max-width: 1200px) {
  .v-application {
    .v-menu__content {
      max-height: 80vh !important;
    }
  }
}
</style>
