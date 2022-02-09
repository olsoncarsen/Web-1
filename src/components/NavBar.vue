<script setup>
import { computed, ref } from 'vue'
import { useStore } from 'vuex'
import {
  mdiForwardburger,
  mdiBackburger,
  mdiClose,
  mdiDotsVertical,
  mdiMenu,
  mdiClockOutline,
  mdiCloud,
  mdiCrop,
  mdiBrightness6
} from '@mdi/js'
import NavBarItem from '@/components/NavBarItem.vue'
import NavBarItemLabel from '@/components/NavBarItemLabel.vue'
import NavBarMenu from '@/components/NavBarMenu.vue'
// import NavBarMenuDivider from '@/components/NavBarMenuDivider.vue'
// import UserAvatar from '@/components/UserAvatar.vue'
import Icon from '@/components/Icon.vue'
import NavBarSearch from '@/components/NavBarSearch.vue'

const store = useStore()

const toggleLightDark = () => {
  store.dispatch('darkMode')
}
const localization = computed(() => store.state.localization.localization)

const isNavBarVisible = computed(() => !store.state.isFullScreen)

const isAsideMobileExpanded = computed(() => store.state.isAsideMobileExpanded)
const navbarLabel = computed(() => store.state.navbar.label)

// const userName = computed(() => store.state.userName)

const menuToggleMobileIcon = computed(() => isAsideMobileExpanded.value ? mdiBackburger : mdiForwardburger)

const menuToggleMobile = () => {
  store.dispatch('asideMobileToggle')
}

const isMenuNavBarActive = ref(false)

const menuNavBarToggleIcon = computed(() => isMenuNavBarActive.value ? mdiClose : mdiDotsVertical)

const menuNavBarToggle = () => {
  isMenuNavBarActive.value = !isMenuNavBarActive.value
}

const menuOpenLg = () => {
  store.dispatch('asideLgToggle', true)
}
</script>

<template>
  <nav
    v-show="isNavBarVisible"
    class="top-0 left-0 right-0 fixed flex bg-white h-14 border-b border-gray-100 z-30 w-screen
    transition-position xl:pl-80 xl:pr-96 lg:w-auto lg:items-stretch dark:bg-gray-900 dark:border-gray-800"
    :class="{'ml-80 lg:ml-80':isAsideMobileExpanded}"
  >
    <div class="flex-1 items-stretch flex h-14">
      <nav-bar-item
        type="flex lg:hidden"
        @click.prevent="menuToggleMobile"
      >
        <icon
          :path="menuToggleMobileIcon"
          size="24"
        />
      </nav-bar-item>
      <nav-bar-item
        type="hidden lg:flex xl:hidden"
        @click.prevent="menuOpenLg"
      >
        <icon
          :path="mdiMenu"
          size="24"
        />
      </nav-bar-item>
      <nav-bar-item>
        {{ navbarLabel }}
      </nav-bar-item>
      <nav-bar-item>
        <nav-bar-search :placeholder="localization.search" />
      </nav-bar-item>
    </div>
    <div class="flex-none items-stretch flex h-14 lg:hidden">
      <nav-bar-item @click.prevent="menuNavBarToggle">
        <icon
          :path="menuNavBarToggleIcon"
          size="24"
        />
      </nav-bar-item>
    </div>
    <div
      class="absolute w-screen top-14 left-0 bg-white shadow
        lg:w-auto lg:items-stretch lg:flex lg:grow lg:static lg:border-b-0 lg:overflow-visible lg:shadow-none dark:bg-gray-900"
      :class="[isMenuNavBarActive ? 'block' : 'hidden']"
    >
      <div class="max-h-screen-menu overflow-y-auto lg:overflow-visible lg:flex lg:items-stretch lg:justify-end lg:ml-auto">
        <nav-bar-item
          has-divider
          is-desktop-icon-only
          @click.prevent="toggleLightDark"
        >
          <nav-bar-item-label
            :icon="mdiBrightness6"
            label="Light/Dark"
            is-desktop-icon-only
          />
        </nav-bar-item>
        <nav-bar-menu has-divider>
          <nav-bar-item-label
            label="Функции"
          />
          <template #dropdown>
            <nav-bar-item>
              <nav-bar-item-label
                :icon="mdiClockOutline"
                label="Show completed tasks"
              />
            </nav-bar-item>
            <nav-bar-item>
              <nav-bar-item-label
                :icon="mdiCloud"
                label="Sort by term"
              />
            </nav-bar-item>
            <nav-bar-item>
              <nav-bar-item-label
                :icon="mdiCrop"
                label="Sort by color"
              />
            </nav-bar-item>
            <nav-bar-item>
              <nav-bar-item-label
                :icon="mdiCrop"
                label="Sort by create date"
              />
            </nav-bar-item>
            <nav-bar-item>
              <nav-bar-item-label
                :icon="mdiCrop"
                label="Print task list"
              />
            </nav-bar-item>
          </template>
        </nav-bar-menu>
        <!-- <nav-bar-menu has-divider>
          <user-avatar class="w-6 h-6 mr-3 inline-flex" />
          <div>
            <span>{{ userName }}</span>
          </div>

          <template #dropdown>
            <nav-bar-item to="/profile">
              <nav-bar-item-label
                :icon="mdiAccount"
                label="My Profile"
              />
            </nav-bar-item>
            <nav-bar-item>
              <nav-bar-item-label
                :icon="mdiCogOutline"
                label="Settings"
              />
            </nav-bar-item>
            <nav-bar-item>
              <nav-bar-item-label
                :icon="mdiEmail"
                label="Messages"
              />
            </nav-bar-item>
            <nav-bar-menu-divider />
            <nav-bar-item>
              <nav-bar-item-label
                :icon="mdiLogout"
                label="Log Out"
              />
            </nav-bar-item>
          </template>
        </nav-bar-menu> -->
        <!-- <nav-bar-item
          href="https://github.com/justboil/admin-one-vue-tailwind"
          has-divider
          is-desktop-icon-only
        >
          <nav-bar-item-label
            :icon="mdiGithub"
            label="GitHub"
            is-desktop-icon-only
          />
        </nav-bar-item> -->
        <!-- <nav-bar-item is-desktop-icon-only>
          <nav-bar-item-label
            :icon="mdiLogout"
            label="Log out"
            is-desktop-icon-only
          />
        </nav-bar-item> -->
      </div>
    </div>
  </nav>
</template>