<template>
  <div class="container d-flex flex-wrap">
    <div class="col-12 col-md-6 p-3">
      <div class="border rounded p-3">
        <h2>操作</h2>
        <form @submit.prevent>
          <BFormControls
              class="mb-3"
              label="名字"
              v-model="formDate.name"/>

          <BFormControls
              class="mb-3"
              label="年齡"
              type="number"
              v-model="formDate.age"/>

          <div class="d-flex gap-1">
            <button class="btn btn-success" @click="edit">修改</button>
            <button class="btn btn-warning" @click="create">新增</button>
          </div>
        </form>
      </div>
    </div>

    <div class="col-12 col-md-6 p-3">
      <div class="border rounded p-3">
        <table class="table">
          <thead>
          <tr>
            <th scope="col">#</th>
            <th scope="col">名字</th>
            <th scope="col">年齡</th>
            <th scope="col">操作</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="(v,i) in users" :key="`user_${i}`">
            <th scope="row">{{ v.id }}</th>
            <td>{{ v.name }}</td>
            <td>{{ v.age }}</td>
            <td class="d-flex gap-1">
              <button class="btn btn-success" @click="selectUser(v)">
                修改
              </button>
              <button class="btn btn-danger" @click="remove(v)">
                刪除
              </button>
            </td>
          </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import axios from 'axios'
import {onMounted, ref} from 'vue';
import BFormControls from "./components/BFormControls.vue";

interface User {
  id: number;
  name: string;
  age: number;
}

const baseUrl = '' // 由面試官提供
const users = ref<User[]>([])
const formDate = ref({
  // id readonly
  id: 0,
  name: '',
  age: 0,
})

const create = () => {
  // 需有確認步驟
}

const edit = () => {
  // 需有確認步驟
}


const selectUser = (user: User) => {
  // 禁止使用 formDate.value = user
}

const remove = (user: User) => {
  // 需有確認步驟
}

const getUsers = () => {
  axios({
    method: 'get',
    url: baseUrl + '/api/user',
  }).then(res => {
    const {data} = res.data
    users.value = data
  }).catch(err => {
    console.log(err)
  })
}

const setupPage = () => {
  getUsers()
}

onMounted(setupPage)
</script>

<style scoped>

</style>
