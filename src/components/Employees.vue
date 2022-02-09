<script setup>
import Icon from '@/components/Icon.vue'
import { mdiCog } from '@mdi/js'
import { useStore } from 'vuex'
import * as TASK from '@/store/actions/tasks'

defineProps({
  employees: {
    type: Array,
    default: () => []
  }
})

const store = useStore()

// Serves as linkage between requests from storage and tree view navigator
const UID_TO_ACTION = {
  'd28e3872-9a23-4158-aea0-246e2874da73': TASK.EMPLOYEE_TASKS_REQUEST
}

const clickOnGridCard = (value) => {
  console.log(value)
  if (UID_TO_ACTION[value.parentID]) {
    if (value.email) {
      if (UID_TO_ACTION[value.parentID] === TASK.EMPLOYEE_TASKS_REQUEST) {
        store.dispatch(UID_TO_ACTION[value.parentID], value.uid)
        console.log('is employee request')
      } else {
        store.dispatch(UID_TO_ACTION[value.parentID], value.email)
        console.log('isnt employee request')
      }
    } else {
      store.dispatch(UID_TO_ACTION[value.parentID], value.uid)
    }
  }
  store.commit('basic', { key: 'mainSectionState', value: 'tasks' })
  store.commit('updateLabel', value.name)
  store.commit(TASK.CLEAN_UP_LOADED_TASKS)
}

</script>

<template class="w-full">
  <div v-for="(value, index) in employees" :key="index">
    <p class="text-3xl text-gray-400 underline font-extralight my-10 dark:text-gray-100">{{ value.dep }}</p>
    <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4 truncate">
      <template v-for="(employee, pindex) in value.items" :key="pindex">
        <div
          class="flex items-start bg-white dark:bg-gray-700 rounded-xl shadow-md hover:shadow-lg cursor-pointer h-30 p-3"
        >
          <img v-if="employee.fotolink" :src="employee.fotolink" class="rounded-lg mx-2 my-auto" width="32" height="32">
          <div class="w-full">
            <div class="flex items-start justify-between">
              <p
                class="font-light cursor-pointer"
                @click="clickOnGridCard(employee)"
              >
                {{ employee.name }}
              </p>
              <icon
                :path="mdiCog"
                size="18"
                class="text-gray-400 cursor-pointer hover:text-gray-800"
              />
            </div>
            <p class="font-light text-xs">{{ employee.email }}</p>
           </div>
        </div>
      </template>
    </div>
  </div>
</template>