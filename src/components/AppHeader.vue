<script setup lang="ts">
import { ref, computed } from 'vue'
import { useUserStore } from '@/stores/user'
import type { ComputedRef } from 'vue'
import { getAuth, signOut } from 'firebase/auth'
import { useRouter } from 'vue-router'

const userStore = useUserStore()
const router = useRouter()

interface IMenuItem {
  label: string
  icon: string
  path: string
  show: ComputedRef<boolean>
}

const items = ref<IMenuItem[]>([
  {
    label: 'Авторизация',
    icon: 'pi pi-user',
    path: '/auth',
    show: computed((): boolean => !userStore.userId)
  },
  {
    label: 'Добавить',
    icon: 'pi pi-plus',
    path: '/',
    show: computed((): boolean => !!userStore.userId)
  },
  {
    label: 'Список заявок',
    icon: 'pi pi-list',
    path: '/list',
    show: computed((): boolean => !!userStore.userId)
  }
])

const signOutMethod = async (): Promise<void> => {
  await signOut(getAuth())
  router.push('/auth')
}
</script>

<template>
  <app-menubar :model="items" class="menu">
    <template #start>
      <svg
        width="35"
        height="40"
        viewBox="0 0 35 40"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
        class="h-8"
      >
        <path d="..." fill="var(--p-primary-color)" />
        <path d="..." fill="var(--p-text-color)" />
      </svg>
    </template>
    <template #item="{ item, props }">
      <template v-if="item.show">
        <router-link :to="item.path" class="flex align-items-center" v-bind="props.action">
          <span :class="item.icon" class="p-menuitem-icon" />
          <span class="ml-2">{{ item.label }}</span>
        </router-link>
      </template>
    </template>
    <template #end>
      <span
        v-if="userStore.userId"
        @click="signOutMethod"
        class="flex align-items-center menu-exit"
      >
        <span class="pi pi-sign-out p-menuitem-icon" />
        <span class="ml-2">Выход</span>
      </span>
    </template>
  </app-menubar>
</template>

<style scoped>
.menu {
  margin: 30px 0;
}
.menu-exit {
  cursor: pointer;
}
</style>
