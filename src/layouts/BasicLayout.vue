<template>
  <a-layout :class="['layout', AppModule.device]">
    <!-- SiderMenu mobile-->
    <a-drawer
      v-if="isMobile()"
      placement="left"
      :wrapClassName="`drawer-sider ${AppModule.navTheme}`"
      :closable="false"
      :visible="collapsed"
      @close="collapsed = false"
    >
      <a-layout-sider
        :class="[
          'sider',
          isDesktop() ? null : 'shadow',
          AppModule.navTheme,
          AppModule.fixSiderbar ? 'ant-fixed-sidemenu' : null
        ]"
        width="256px"
        :collapsible="true"
        :trigger="null"
      >
        <sider-menu
          mode="inline"
          :collapsed="false"
          @handleClick="handleClick"
        />
      </a-layout-sider>
    </a-drawer>
    <!-- SiderMenu -->
    <a-layout-sider
      v-else-if="isSideMenu()"
      :class="[
        'sider',
        isDesktop() ? null : 'shadow',
        AppModule.navTheme,
        AppModule.fixSiderbar ? 'ant-fixed-sidemenu' : null
      ]"
      width="256px"
      :collapsible="true"
      v-model="collapsed"
      :trigger="null"
    >
      <sider-menu :collapsed="collapsed" mode="inline" />
    </a-layout-sider>

    <a-layout
      :class="[AppModule.layoutMode, `content-width-${AppModule.contentWidth}`]"
      :style="{ paddingLeft: contentPaddingLeft, minHeight: '100vh' }"
    >
      <!-- layout global header -->
      <global-header :collapsed="collapsed" @toggle="toggle" />

      <!-- layout content -->
      <a-layout-content
        :style="{
          height: '100%',
          margin: '24px 24px 0',
          paddingTop: AppModule.fixedHeader ? '64px' : '0'
        }"
      >
        <!-- <multi-tab v-if="multiTab"></multi-tab> -->
        <transition name="page-transition">
          <router-view />
        </transition>
      </a-layout-content>

      <!-- layout footer -->
      <a-layout-footer>
        <global-footer />
      </a-layout-footer>

      <!-- Setting Drawer -->
      <!-- 权限组件使用方式 -->
      <!-- <Authorized :authority="['admin']">
       <setting-drawer />
      </Authorized> -->
      <!-- 权限指令使用方式 -->
      <setting-drawer v-auth="['admin']" />
    </a-layout>
  </a-layout>
</template>

<script lang="ts">
import { Vue, Component, Mixins, Watch } from 'vue-property-decorator'
import { Mixin, DeviceMixin } from '@/utils/mixins'
import { triggerWindowResizeEvent } from '@/utils/helper/'
import config from '@/config/defaultSettings'
import GlobalHeader from './GlobalHeader.vue'
import GlobalFooter from './GlobalFooter.vue'
import SiderMenu from './menu/SiderMenu.vue'
import { SettingDrawer } from '@/components'

@Component({
  components: {
    GlobalHeader,
    GlobalFooter,
    SiderMenu,
    SettingDrawer
  }
})
export default class BasicLayout extends Mixins(Mixin, DeviceMixin) {
  private collapsed: boolean = false

  private created() {
    console.log(this.AppModule)
    this.collapsed = !this.AppModule.sidebar
  }

  get contentPaddingLeft() {
    if (!this.AppModule.fixSiderbar || this.isMobile()) {
      return '0'
    }
    if (this.AppModule.sidebar) {
      return '256px'
    }
    return '80px'
  }

  private toggle() {
    this.collapsed = !this.collapsed
    this.AppModule.SetSidebar(!this.collapsed)
    triggerWindowResizeEvent()
  }

  private handleClick({ item, key, keyPath }) {
    if (!this.isDesktop()) {
      this.collapsed = false
    }
  }

  @Watch('AppModule.sidebar')
  sidebarChange(val) {
    this.collapsed = !val
  }
}
</script>

<style lang="less">
@import url('../assets/styles/global.less');
.page-transition-enter {
  opacity: 0;
}

.page-transition-leave-active {
  opacity: 0;
}

.page-transition-enter .page-transition-container,
.page-transition-leave-active .page-transition-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
</style>
