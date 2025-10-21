<template>
  <a-row :wrap="false">
    <a-col flex="200px">
      <RouterLink to="/">
        <div class="title-bar">
          <img class="logo" src="../assets/logo.svg" alt="logo" />
          <div class="title">kui的云图库</div>
        </div>
      </RouterLink>
    </a-col>
    <a-col flex="auto">
      <a-menu
        v-model:selectedKeys="current"
        mode="horizontal"
        :items="items"
        @click="doMenuClick"
      />
    </a-col>
    <a-col flex="120px">
      <div class="user-login-status">
        <div v-if="loginUserStore.loginUser">
          {{ loginUserStore.loginUser.userName ?? '未设定昵称' }}
        </div>
        <div v-else>
          <a-button type="primary" href="/user/login">登录</a-button>
        </div>
      </div>
    </a-col>
  </a-row>
</template>
<script lang="ts" setup>
import { computed, h, ref } from 'vue'
import { HomeOutlined } from '@ant-design/icons-vue'
import router from '@/router'
import { useLoginUserStore } from '@/stores/user.ts'
import type { MenuProps } from 'ant-design-vue'

const loginUserStore = useLoginUserStore()

// 当前选中菜单
const current = ref<string[]>([])
// 监听路由变化，更新当前选中菜单
router.afterEach((to, from, next) => {
  current.value = [to.path]
})

const items = computed<MenuProps['items']>(() => {
  return router.options.routes.map((route) => ({
    key: route.path,
    label: route.meta?.title,
    icon: route.meta?.icon ? () => h(route.meta!.icon!) : undefined,
  }))
})
const doMenuClick = ({ key }: { key: string }) => {
  router.push({
    path: key,
  })
}
</script>

<style scoped>
.title-bar {
  display: flex;
  align-items: center;
}

.title {
  color: black;
  font-size: 18px;
  margin-left: 16px;
}

.logo {
  height: 48px;
}
</style>
