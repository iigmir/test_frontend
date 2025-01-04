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
import axios from "axios";
import { onMounted, ref } from "vue";
import BFormControls from "./components/BFormControls.vue";

// User module
interface User {
  id: number;
  name: string;
  age: number;
}
const users = ref<User[]>([]);
const formDate = ref({
  // id readonly
  id: 0,
  name: '',
  age: 0,
});
const setFormDate = (user: User) => {
  // 禁止使用 formDate.value = user
  formDate.value.id = user.id;
  formDate.value.name = user.name;
  formDate.value.age = user.age;
};
const resetFormDate = () => {
  // 禁止使用 formDate.value = user
  setFormDate({
    id: 0,
    name: "",
    age: 0
  });
}

// API動作
const baseUrl = 'https://oks96yif.wuc.us.kg'; // 由面試官提供
const apiPath = `${baseUrl}/api/user`;

// 驗證步驟
const idIsPassed = (input: unknown): boolean => {
  // No, we don't accept string, either
  if (typeof input !== "number") {
    return false;
  }
  return isNaN( input ) === false && input > 0;
};
const nameIsPassed = (input: unknown): boolean => {
  if( typeof input !== "string" ) {
    return false;
  }
  return input.trim() !== "";
};
const ageIsPassed = (input: unknown): boolean => {
  // No, we don't accept string, either
  if (typeof input !== "number") {
    return false;
  }
  return isNaN( input ) === false && input > 0;
};
const formDateValidator = (input: User) => {
  return nameIsPassed(input.name) && ageIsPassed(input.age);
};

// 確認步驟
const confirmChoice = (input: User, question = "") => {
  return new Promise( (resolve, reject) => {
    const answer = window.confirm(question);
    if( answer ) {
      resolve(input);
    } else {
      reject(input);
    }
  });
};
const actionCanceled = () => {
  alert("動作已中止。");
};

// Client side CRUD module
// Methods there must execute AFTER finishing methods at server side CRUD module
const searchingItemById = (input: User) => users.value.findIndex( item => item.id === input.id );
const createClientAction = ({ data } = { data: { id: 0 } }, fillingFormDate: User) => {
  users.value.push({
    id: data.id,
    name: fillingFormDate.name,
    age: fillingFormDate.age
  });
  resetFormDate();
};
const editClientAction = ({ data } = { data: { id: 0 } }, fillingFormDate: User) => {
  const index = searchingItemById(fillingFormDate);
  console.log(data);
  users.value[index] = {
    id: fillingFormDate.id,
    name: fillingFormDate.name,
    age: fillingFormDate.age
  };
};
const removeClientAction = () => {};

// Server side CRUD module
const create = () => {
  // 需有確認步驟
  const passed = formDateValidator(formDate.value);
  if( passed ) {
    const confirm = confirmChoice(
      formDate.value,
      `確定新增 ${formDate.value.name}(${formDate.value.age}) 嗎？`
    );
    confirm.then( () => {
      axios({
        method: "POST",
        url: apiPath,
        data: {
          age: formDate.value.age,
          name: formDate.value.name,
        }
      }).then( res => {
        createClientAction(res.data, formDate.value)
      }).catch(err => {
        alert("請求失敗！");
        console.log(err);
      });
    }).catch( actionCanceled );
    return;
  } else {
    alert("請輸入姓名及年齡。");
  }
};
const edit = () => {
  const newLocal = idIsPassed(formDate.value.id);
  // 需有確認步驟
  const passed = formDateValidator(formDate.value) && newLocal;
  if( passed ) {
    const confirm = confirmChoice(
      formDate.value, 
      `確定修改 ${formDate.value.name}(${formDate.value.age}) 嗎？`
    );
    confirm.then( (req) => {
      axios({
        method: "PUT",
        url: apiPath,
        data: {
          id: formDate.value.id,
          age: formDate.value.age,
          name: formDate.value.name,
        }
      }).then( (res) => {
        editClientAction(res.data, formDate.value)
      }).catch( (err) => {
        alert("請求失敗！");
        console.log(err);
      });
      console.log(req);
    }).catch( actionCanceled );
    return;
  } else if( idIsPassed(formDate.value.id) === false ) {
    create();
    return;
  } else {
    alert("請輸入姓名及年齡。");
  }
};
const remove = (user: User) => {
  // 需有確認步驟
  const passed = idIsPassed(user.id);
  if( passed ) {
    const confirm = confirmChoice(
      formDate.value,
      "確定刪除嗎？"
    );
    confirm.then( (req) => {
      console.log(req);
      removeClientAction();
    }).catch( actionCanceled );
    return;
  } else {
    alert("請求刪除的資料無效。");
  }
};

const selectUser = (user: User) => {
  // 禁止使用 formDate.value = user
  setFormDate(user);
};

const getUsers = () => {
  axios({
    method: "GET",
    url: apiPath,
  }).then( res => {
    const { data } = res.data;
    users.value = data;
  }).catch(err => {
    console.log(err)
  });
};

const setupPage = () => {
  getUsers();
};

onMounted(setupPage);
</script>

<style scoped>

</style>
