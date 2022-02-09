<script setup>
import { computed, reactive, ref } from 'vue'
import { useStore } from 'vuex'
import { useRouter } from 'vue-router'
import ModalBox from '@/components/ModalBox.vue'
import { DatePicker } from 'v-calendar'
import { mdiMenu } from '@mdi/js'
import JbButton from '@/components/JbButton.vue'
import NavBarItem from '@/components/NavBarItem.vue'
import Icon from '@/components/Icon.vue'

import AsideMenuList from '@/components/AsideMenuList.vue'
import iconmenu from '@/icons/menu.js'

import * as TASK from '@/store/actions/tasks'
import { AUTH_LOGOUT } from '@/store/actions/auth'

const router = useRouter()

defineProps({
  menu: {
    type: Array,
    default: () => []
  }
})

let modalOneActive = ref(false)

// Serves as linkage between requests from storage and tree view navigator
const UID_TO_ACTION = {
  '901841d9-0016-491d-ad66-8ee42d2b496b': TASK.TASKS_REQUEST, // get today's day
  '46418722-a720-4c9e-b255-16db4e590c34': TASK.OVERDUE_TASKS_REQUEST,
  '017a3e8c-79ac-452c-abb7-6652deecbd1c': TASK.OPENED_TASKS_REQUEST,
  '5183b619-3968-4c3a-8d87-3190cfaab014': TASK.UNSORTED_TASKS_REQUEST,
  'fa042915-a3d2-469c-bd5a-708cf0339b89': TASK.UNREAD_TASKS_REQUEST,
  '2a5cae4b-e877-4339-8ca1-bd61426864ec': TASK.IN_WORK_TASKS_REQUEST,
  '6fc44cc6-9d45-4052-917e-25b1189ab141': TASK.IN_FOCUS_TASKS_REQUEST,
  'd35fe0bc-1747-4eb1-a1b2-3411e07a92a0': TASK.READY_FOR_COMPLITION_TASKS_REQUEST,
  '169d728b-b88b-462d-bd8e-3ac76806605b': TASK.DELEGATED_TASKS_REQUEST,
  '511d871c-c5e9-43f0-8b4c-e8c447e1a823': TASK.DELEGATED_TO_USER_TASKS_REQUEST,
  '7af232ff-0e29-4c27-a33b-866b5fd6eade': TASK.PROJECT_TASKS_REQUEST, // private
  '431a3531-a77a-45c1-8035-f0bf75c32641': TASK.PROJECT_TASKS_REQUEST, // shared
  'd28e3872-9a23-4158-aea0-246e2874da73': TASK.EMPLOYEE_TASKS_REQUEST,
  'ed8039ae-f3de-4369-8f32-829d401056e9': TASK.COLOR_TASKS_REQUEST,
  '00a5b3de-9474-404d-b3ba-83f488ac6d30': TASK.TAG_TASKS_REQUEST
}

const store = useStore()
const isFullScreen = computed(() => store.state.isFullScreen)
const isAsideMobileExpanded = computed(() => store.state.isAsideMobileExpanded)
const isAsideLgActive = computed(() => store.state.isAsideLgActive)
const isDark = computed(() => store.state.darkMode)

const datePickerBG = computed(() => {
  return isDark.value ? 'rgb(51 65 85)' : '#fff7ed'
})

const localization = computed(() => store.state.localization.localization)
const attrs = computed(() => store.state.calendar.calendar)
const user = computed(() => store.state.user.user)
const storeNavigator = computed(() => store.state.navigator.navigator)

const getNavigatorLanguage = () => (navigator.languages && navigator.languages.length) ? navigator.languages[0] : navigator.userLanguage || navigator.language || navigator.browserLanguage || 'en'

const currentDate = computed({
  get: () => new Date(),
  set: val => {
    store.commit('basic', { key: 'mainSectionState', value: 'tasks' })
    store.dispatch(TASK.TASKS_REQUEST, val)
    store.commit('updateLabel', dateToLabelFormat(val))
    store.commit(TASK.CLEAN_UP_LOADED_TASKS)
  }
})

const navigatorMenu = reactive({
  tasks: false,
  foldableNavigator: false,
  lang: getNavigatorLanguage(),
  currentDate: currentDate
})

const logout = () => {
  modalOneActive = false
  store.dispatch(AUTH_LOGOUT)
  router.push('/login')
}

function dateToLabelFormat (calendarDate) {
  const day = calendarDate.getDate()
  const month = calendarDate.toLocaleString('default', { month: 'short' })
  const weekday = calendarDate.toLocaleString('default', { weekday: 'short' })
  return day + ' ' + month + ', ' + weekday
}

const asideLgClose = () => {
  store.dispatch('asideLgToggle', false)
}

const menuClick = (event, item) => {
  store.commit('updateLabel', item.label)
  if (UID_TO_ACTION[item.uid] && item.type === 'uid') {
    store.dispatch(UID_TO_ACTION[item.uid])
    store.commit('basic', { key: 'mainSectionState', value: 'tasks' })
  } else {
    store.commit('basic', { key: 'mainSectionState', value: 'greed' })
    store.commit('basic', { key: 'greedPath', value: item.path })
    if (item.path === 'new_private_projects' || item.path === 'new_emps') {
      store.commit('basic', { key: 'greedSource', value: storeNavigator.value[item.path] })
      console.log(storeNavigator.value[item.path])
    } else {
      store.commit('basic', { key: 'greedSource', value: storeNavigator.value[item.path].items })
    }
    console.log(storeNavigator.value[item.path])
  }
  store.commit(TASK.CLEAN_UP_LOADED_TASKS)
}
</script>

<template>
  <!-- Profile modal -->
  <modal-box
    v-model="modalOneActive"
    title="Profile"
  >
    <img
      :src="user.foto_link"
      class="mx-auto rounded-full"
    >
    <strong>Current user: </strong>
    <br>
    <span>{{ user.current_user_name }}</span>
    <br>
    <span>{{ user.current_user_email }}</span>
    <br>
    <br>
    <strong>{{ localization.owner_license }}</strong>
    <br>
    <span>{{ user.owner_title }}</span>
    <br>
    <span>{{ user.owner_email }}</span>
    <br>
    <br>
    <strong>License expires</strong>
    <br>
    <span>{{ user.date_expired }}</span>
    <br>
    <br>
    <strong>Days left</strong>
    <br>
    <span>{{ user.days_left }}</span>
    <br>
    <br>
    <strong>Used server storage </strong>
    <span> {{ user.total_mb }} {{ localization.megabytes }} ({{ user.percent_mb }}%)</span>
    <br>
    <br>
    <jb-button
      label="Logout"
      color="danger"
      @click="modalOneActive = false; logout()"
    />
  </modal-box>
  <!-- /Profile modal -->

  <aside
    v-show="!isFullScreen"
    id="aside"
    class="w-80 fixed top-0 z-40 h-screen bg-white transition-position lg:left-0 dark:border-r dark:border-gray-800 dark:bg-gray-900"
    :class="[ isAsideMobileExpanded ? 'left-0' : '-left-80', isAsideLgActive ? 'block' : 'lg:hidden xl:block' ]"
  >
    <div class="flex flex-row w-full text-dark flex-1 h-12 items-center bg-orange-50 dark:bg-slate-700">
      <nav-bar-item
        type="hidden lg:flex xl:hidden"
        active-color="text-dark"
        active
        @click="asideLgClose"
      >
        <icon
          :path="mdiMenu"
          class="cursor-pointer"
          size="24"
        />
      </nav-bar-item>
      <div
        class="flex justify-between items-center w-full px-3 cursor-pointer"
        @click="modalOneActive = true"
      >
        <icon
          :path="iconmenu.path"
          class="cursor-pointer"
          :box="iconmenu.viewBox"
          :width="iconmenu.width"
          :height="iconmenu.height"
        />
        <div class="flex">
          <span
            class="font-light text-sm my-auto mr-2"
          >
            {{ user.current_user_name }}
          </span>
          <img
            :src="user.foto_link"
            width="32"
            height="32"
            class="rounded-lg"
          >
        </div>
      </div>
    </div>
    <nav-bar-item class="bg-orange-50 dark:bg-slate-700 rounded-b-3xl pt-0 mt-0">
      <DatePicker
        v-model="navigatorMenu.currentDate"
        class="border-none bg-orange-50 dark:bg-slate-700 pb-5 text-xs"
        style="border: none!important;"
        :style="{ backgroundColor: datePickerBG }"
        show-weeknumbers
        color="yellow"
        is-expanded
        :locale="navigatorMenu.lang"
        :attributes="attrs"
        :is-dark="isDark"
       />
    </nav-bar-item>
    <div class="my-5">
      <template v-for="(menuGroup, index) in menu">
        <div
          v-if="typeof menuGroup === 'string'"
          :key="`a-${index}`"
          class="my-2"
        >
          <hr
            :key="`a-${index}`"
            class="text-xs mx-3"
            :class="[ asideMenuLabelStyle ]"
          >
        </div>
        <aside-menu-list
          v-else
          :key="`b-${index}`"
          :menu="menuGroup"
          @menu-click="menuClick"
        />
      </template>
    </div>
  </aside>
</template>

<style>
  .navigator-tree .e-list-text {
    font-size: 16px!important;
  }
  .navigator-tree .e-ul, .navigator-tree .e-text-content{
    padding-left: 4px!important;
  }
  .navigator-tree .e-icon-collapsible, .navigator-tree .e-icon-expandable {
    float: right;
    margin-right: 20px
  }
  .navigator-tree .e-level-1 > .e-text-content .e-list-text {
    color: gray;
    padding-left: 0px;
  }
  .navigator-tree .e-level-2 > .e-text-content {
    padding-left: 0px;
  }
  .vc-weeks {
    padding: 0;
  }
  .vc-arrow {
    background-color: rgba(255, 145, 35, 0.15);
    border-radius: 10px;
    color: #2A2927;
  }
  .vc-weeknumber-content {
    @apply bg-white dark:bg-gray-800 rounded-lg text-violet-600 dark:text-white;
  }
  .vc-focusable {
    @apply rounded-lg;
  }
  .vc-day-layer {
    @apply rounded-lg;
  }
  .vc-title {
    color: #2A2927;
  }
  .vc-weekday {
    color: #2A2927;
  }
  .vc-header {
    margin-bottom: 10px;
  }
  .vc-arrows-container {
    padding-left: 0;
    padding-right: 0;
  }
  .vc-container .vc-day-content:hover:not(.is-disabled) {
    @apply bg-gray-400 text-white;
  }
  .vc-container .vc-day-content.is-disabled {
    @apply pointer-events-none;
  }
  .vc-container .vc-day-content.is-disabled:hover {
    @apply bg-transparent;
  }
  .vc-day.is-not-in-month *:not(.is-disabled) {
    @apply opacity-100 text-gray-500 pointer-events-auto;
  }
  .vc-day.is-not-in-month .is-disabled  {
    @apply opacity-100 text-gray-400 pointer-events-none;
  }
  .vc-day.weekday-7 {
    @apply text-red-500;
  }
  .vc-day.weekday-1 {
    @apply text-red-500;
  }
  .vc-weekday:nth-last-of-type(-n+2) {
    @apply text-red-500;
  }
</style>