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
        <div v-if="loginUserStore.loginUser.id">
          <a-dropdown>
            <ASpace>
              <a-avatar :src="loginUserStore.loginUser.userAvatar" />
              {{ loginUserStore.loginUser.userName ?? '无名' }}
            </ASpace>
            <template #overlay>
              <a-menu>
                <a-menu-item>
                  <router-link to="/my_space">
                    <UserOutlined />
                    我的空间
                  </router-link>
                </a-menu-item>
                <a-menu-item @click="doLogout">
                  <LogoutOutlined />
                  退出登录
                </a-menu-item>
              </a-menu>
            </template>
          </a-dropdown>
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
import { HomeOutlined, LogoutOutlined, UserOutlined } from '@ant-design/icons-vue'
import router from '@/router'
import { useLoginUserStore } from '@/stores/userLoginUserStore.ts'
import { message } from 'ant-design-vue'
import { userLogoutUsingPost } from '@/api/userController.ts'
import checkAccess from '@/access/checkAccess.ts'
import ACCESS_ENUM from '@/access/accessEnum.ts'
import GithubOutlined from '@ant-design/icons-vue/lib/icons/GithubOutlined'

const loginUserStore = useLoginUserStore()

// 把路由项转换为菜单项
const menuToRouteItem = (item: any) => {
  return {
    key: item.key, // 用于路由的跳转
    label: item.label,
    title: item.title,
    icon: item.icon ?? undefined,
  }
}

// 过滤菜单项
const items = computed(() => {
  return originItems
    .filter((item) => {
      console.log(item.meta?.access)
      // 根据权限过滤菜单，有权限则返回 true，则保留该菜单
      return checkAccess(loginUserStore.loginUser, item.meta?.access as string)
    })
    .map(menuToRouteItem) // 转换为菜单项格式
})

// 当前选中菜单
const current = ref<string[]>([])
// 监听路由变化，更新当前选中菜单
router.afterEach((to, from, next) => {
  current.value = [to.path]
})
// 路由跳转事件
const doMenuClick = ({ key }: { key: string }) => {
  router.push({
    path: key,
  })
}
// 用户注销
const doLogout = async () => {
  const res = await userLogoutUsingPost()
  console.log(res)
  if (res.data.code === 0) {
    loginUserStore.setLoginUser({
      userName: '未登录',
    })
    message.success('退出登录成功')
    await router.push('/user/login')
  } else {
    message.error('退出登录失败，' + res.data.message)
  }
}

// 菜单列表
const originItems = [
  {
    key: '/',
    icon: () => h(HomeOutlined),
    label: '主页',
    title: '主页',
    meta: {
      access: ACCESS_ENUM.NOT_LOGIN,
    },
  },
  {
    key: '/add_picture',
    label: '创建图片',
    title: '创建图片',
    meta: {
      access: ACCESS_ENUM.USER,
    },
  },
  {
    key: '/add_picture/batch',
    label: '批量创建图片',
    title: '批量创建图片',
    meta: {
      access: ACCESS_ENUM.USER,
    },
  },
  {
    key: '/add_space',
    label: '创建空间',
    title: '创建空间',
    meta: {
      access: ACCESS_ENUM.USER,
    },
  },
  {
    key: '/admin/userManage',
    label: '用户管理',
    title: '用户管理',
    meta: {
      access: ACCESS_ENUM.ADMIN,
    },
  },
  {
    key: '/admin/pictureManage',
    label: '图片管理',
    title: '图片管理',
    meta: {
      access: ACCESS_ENUM.ADMIN,
    },
  },
  {
    key: '/admin/spaceManage',
    label: '空间管理',
    title: '空间管理',
    meta: {
      access: ACCESS_ENUM.ADMIN,
    },
  },
  {
    key: '/about',
    label: '关于',
    title: '关于',
    meta: {
      access: ACCESS_ENUM.NOT_LOGIN,
    },
  },
  {
    key: 'kuiGithub',
    icon: () => h(GithubOutlined),
    label: h('a', { href: 'https://github.com/kui23331', target: '_blank' }, 'kui的GitHub主页'),
    title: 'kui233_GitHub',
    meta: {
      access: ACCESS_ENUM.NOT_LOGIN,
    },
  },
]
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
