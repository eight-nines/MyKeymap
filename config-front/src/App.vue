<template>
  <v-app v-if="$store.state.config">
    <v-navigation-drawer app id="drawer" permanent>
      <v-list-item>
        <v-list-item-avatar rounded="0" class="logo">
          <v-img alt="img" :src="require('./assets/logo.png')"></v-img>
        </v-list-item-avatar>
        <v-list-item-content>
          <v-list-item-title id="site-title"> MyKeymap </v-list-item-title>
          <v-list-item-subtitle> version: 1.2.4 </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>

      <v-divider></v-divider>

      <v-list nav>
        <v-list-item class="nav-item" v-for="item in enabledItems" :key="item.title" :to="{ name: item.to }">
          <v-list-item-icon>
            <v-icon class="nav-tab-icon" :color="item.color">{{ item.icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-divider></v-divider>
        <v-btn width="100%" elevation="0" color="primary" @click="saveConfig" outlined>
          <v-icon left> mdi-content-save </v-icon> 保存配置 (ctrl+s)</v-btn
        >
      </v-list>
    </v-navigation-drawer>

    <v-main id="main">
      <HelpPage v-show="false" />
      <v-card v-if="!currentModeEnabled" outlined max-width="100%" dark id="warn" color="#555">
        <v-card-title>此模式尚未开启, 若想使用需要在设置中打开</v-card-title>
      </v-card>
      <router-view v-if="$store.state.config" />
    </v-main>

    <v-snackbar id="snack-bar" v-model="$store.state.snackbar" color="purple" timeout="1600" min-width="600" height="70">
      <span id="snackBarText">{{ $store.state.snackbarText }}</span>
      <template v-slot:action="{ attrs }">
        <v-btn color="black" text v-bind="attrs" @click="snackbar = false"> </v-btn>
      </template>
    </v-snackbar>
  </v-app>
</template>

<script>
import hotkeys from 'hotkeys-js'
import HelpPage from './components/HelpPage.vue'
import { isModeEnabled } from './util'

// By default hotkeys are not enabled for INPUT SELECT TEXTAREA elements.
hotkeys.filter = function (event) {
  return true
}

export default {
  components: { HelpPage },
  name: 'App',
  created() {
    hotkeys('ctrl+s', (event, handler) => {
      event.preventDefault() // Prevent the default refresh event under WINDOWS system
      this.saveConfig()
    })
  },
  computed: {
    currentModeEnabled() {
      if (!this.$store.state.config) return true
      let mode = this.$route.name
      let settings = this.$store.state.config.Settings;
      return this.isModeEnabled(mode, settings)
    },
    enabledItems() {
      if (!this.$store.state.config) return this.items
      return this.items.filter(x => this.isModeEnabled(x.to, this.$store.state.config.Settings))
    }
  },
  data: () => ({
    items: [
      { title: 'Capslock', icon: 'mdi-alpha-c-box', to: 'Capslock', color: 'purple' },
      { title: 'Capslock + F', icon: 'mdi-alpha-c-box', to: 'CapslockF', color: 'purple' },
      { title: 'Capslock + Space', icon: 'mdi-alpha-c-box', to: 'CapslockSpace', color: 'purple' },
      { title: 'Capslock 指令', icon: 'mdi-rocket-launch', to: 'CapslockAbbr', color: 'deep-orange' },
      { title: '缩写功能', icon: 'mdi-format-text-variant-outline', to: 'SemicolonAbbr', color: 'deep-orange' },
      { title: '自定义热键', icon: 'mdi-keyboard', to: 'CustomHotkeys', color: 'blue' },
      { title: 'Tab 模式', icon: 'mdi-alpha-t-box', to: 'TabMode', color: '#d05' },
      { title: '空格模式', icon: 'mdi-alpha-s-box', to: 'SpaceMode', color: '#d05' },
      { title: 'J 模式', icon: 'mdi-alpha-j-box', to: 'JMode', color: '#d05' },
      { title: 'J + K 模式', icon: 'mdi-alpha-j-box', to: 'JModeK', color: '#d05' },
      { title: '逗号模式', icon: 'mdi-rhombus', to: 'CommaMode', color: '#d05' },
      { title: '句号模式', icon: 'mdi-rhombus', to: 'DotMode', color: 'blue' },
      { title: '分号模式', icon: 'mdi-rhombus', to: 'Semicolon', color: 'blue' },
      { title: '3 模式', icon: 'mdi-numeric-3-box-outline', to: 'Mode3', color: 'red' },
      { title: '9 模式', icon: 'mdi-numeric-9-box-outline', to: 'Mode9', color: 'red' },
      { title: '鼠标左键', icon: 'mdi-cursor-default-outline', to: 'LButtonMode', color: '' },
      { title: '鼠标右键', icon: 'mdi-cursor-default', to: 'RButtonMode', color: '' },
      // { title: '使用说明', icon: 'mdi-help', to: 'About', color: 'light-green' },
      // { title: '关于作者', icon: 'mdi-exclamation-thick', to: 'About', color: 'light-green' },
      { title: '开关/设置', icon: 'mdi-toggle-switch', to: 'Settings', color: 'light-green' },
    ],
  }),
  methods: {
    saveConfig() {
      this.$store.dispatch('saveConfig')
    },
    isModeEnabled,
  },
}
</script>

<style>
body * {
  font-family: Roboto, sans-serif, Microsoft YaHei !important;
}

.v-avatar.logo {
  height: 60px !important;
  width: 60px !important;
}
#drawer {
  box-shadow: 0px 0px 1px 1px rgba(0, 0, 0, 0.1);
}
#main {
  background: #f2f3f6;
  background: #fafafa;
}
.nav-item {
  margin-bottom: 3px !important;
}
.nav-tab-icon {
  font-size: 26px !important;
}
#snackBarText {
  color: white;
  /* font-weight: 600; */
  font-size: 1.5em;
  padding-left: 115px;
}
#site-title {
  font-size: 1.6em;
  font-weight: 500;
}
#snack-bar {
  margin-bottom: 110px;
}

#warn {
  margin: 20px;
  margin-top: 5px;
  margin-bottom: 1px;
}

.v-navigation-drawer__content::-webkit-scrollbar {
  width: 0 !important;
}
.my-container {
  margin: 30px 20px 20px 20px;
}

/* width */
::-webkit-scrollbar {
  width: 5px;
}

/* Track */
::-webkit-scrollbar-track {
  background: #f1f1f1; 
}
 
/* Handle */
::-webkit-scrollbar-thumb {
  background: #999; 
}


</style>