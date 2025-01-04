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

// CRUD module: AJAX & slient render submodule
const searchingItemById = (input: User) => users.value.findIndex( item => item.id === input.id );
const createClientAction = (fillingFormDate: User) => {
  axios({
    method: "POST",
    url: apiPath,
    data: {
      age: fillingFormDate.age,
      name: fillingFormDate.name,
    }
  }).then( res => {
    const { data } = res.data;
    users.value.push({
      id: data.id,
      name: fillingFormDate.name,
      age: fillingFormDate.age
    });
    resetFormDate();
  }).catch(err => {
    alert("請求失敗！");
    console.log(err);
  });
};
const editClientAction = (fillingFormDate: User) => {
  axios({
    method: "PUT",
    url: apiPath,
    data: {
      id: fillingFormDate.id,
      age: fillingFormDate.age,
      name: fillingFormDate.name,
    }
  }).then( (res) => {
    const { data } = res.data;
    const index = searchingItemById(fillingFormDate);
    console.log(data);
    users.value[index] = {
      id: fillingFormDate.id,
      name: fillingFormDate.name,
      age: fillingFormDate.age
    };
  }).catch( (err) => {
    alert("請求失敗！");
    console.log(err);
  });
};
const removeClientAction = (fillingFormDate: User) => {
  axios({
    method: "DELETE",
    url: apiPath,
    data: {
      id: fillingFormDate.id,
    }
  }).then( (res) => {
      const index = searchingItemById(fillingFormDate);
      console.log(res);
      users.value.splice(index, 1);
  }).catch( (err) => {
    alert("請求失敗！");
    console.log(err);
  });
};

// CRUD module: Confirmation submodule
const create = () => {
  // 需有確認步驟
  const passed = formDateValidator(formDate.value);
  if( passed ) {
    const confirm = confirmChoice(
      formDate.value,
      `確定新增 ${formDate.value.name}(${formDate.value.age}) 嗎？`
    );
    confirm.then( () => {
      createClientAction(formDate.value)
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
    confirm.then( () => {
      editClientAction(formDate.value);
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
    confirm.then( () => {
      removeClientAction(formDate.value);
    }).catch( actionCanceled );
    return;
  } else {
    alert("請求刪除的資料無效。");
  }
};

// Rendering module
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
