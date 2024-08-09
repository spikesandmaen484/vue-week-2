<script setup>
import { ref } from "vue";
import axios from 'axios';

const api_url = 'https://todolist-api.hexschool.io';

const isLogin = ref(false);
const isRegister = ref(false);

const email = ref('');
const password = ref('');
const nickname = ref('');
const token = ref('');
const hidToken = ref('');

//切換註冊、登入頁面
const switchPage = () => {
  //清空暫存
  email.value = '';
  password.value = '';
  nickname.value = '';

  isRegister.value = !isRegister.value;
}

//註冊
const signUp = async () => {
  try {
    const res = await axios.post(`${api_url}/users/sign_up`, {
      email: email.value,
      password: password.value,
      nickname: nickname.value
    });

    alert('註冊完成!');
    switchPage();

  } catch (error) {
    alert('註冊失敗:' + error.message);
  }
}

//登入
const signIn = async () => {
  try {
    const res = await axios.post(`${api_url}/users/sign_in`, {
      email: email.value,
      password: password.value
    });

    token.value = res.data.token;
    hidToken.value = res.data.token;
    alert('登入成功! 驗證Token為 ' + token.value);

    //清空暫存
    email.value = '';
    password.value = '';
    isLogin.value = true;

    getTodoList();

  } catch (error) {
    alert('登入失敗:' + error.message);
  }
}

//驗證
const validToken = async () => {
  try {
    const res = await axios.get(`${api_url}/users/checkout`, {
      headers:{
        Authorization: token.value
      }
    });

    alert('驗證成功! uid: ' + token.value);

  } catch (error) {
    alert('驗證錯誤:' + error.message);
  }
}

//登出
const signOut = async () => {
  try {
    const res = await axios.post(`${api_url}/users/sign_out`, {},{
      headers:{
        Authorization: hidToken.value
      }
    });

    alert('登出成功!');

    token.value = '';
    hidToken.value = '';
    isLogin.value = false;

  } catch (error) {
    alert('驗證錯誤:' + error.message);
  }
}

const todoList = ref([]);
const inputTodo = ref('');

//取得TodoList
const getTodoList = async () => {
  try {
    const res = await axios.get(`${api_url}/todos`, {
      headers:{
        Authorization: hidToken.value
      }
    });

    todoList.value = res.data.data.map((item) => ({...item, isEdit: false}));

  } catch (error) {
    alert('取得TodoList失敗:' + error.message);
  }
}

//新增Todo
const addTodo = async () => {
  try {
    const res = await axios.post(`${api_url}/todos`,
      {
        content: inputTodo.value
      },
      {
        headers: {
            Authorization: hidToken.value
          }
    });

    alert('新增Todo完成');
    inputTodo.value = '';
    getTodoList();

  } catch (error) {
    alert('新增Todo失敗:' + error.message);
  }
}

//刪除Todo
const delTodo = async (id) => {
  try {
    const res = await axios.delete(`${api_url}/todos/${id}`, {
      headers: {
        Authorization: hidToken.value
      }
    });

    alert('刪除Todo完成!');
    getTodoList();

  } catch (error) {
    alert('刪除Todo失敗:' + error.message);
  }
}

//編輯Todo
const editTodo = (todo) => {
  if (todo.isEdit) {
    saveTodo(todo.id);
  }
  else {
    todo.isEdit = true;
  }
}

//儲存Todo
const saveTodo = async (id) => {
  try {
    const data = todoList.value.find((item) => item.id === id);

    const res = await axios.put(`${api_url}/todos/${id}`, 
      {
        content: data.content
      },
      {
        headers:{
          Authorization: hidToken.value
        }
    });

    alert('儲存Todo成功!');
    getTodoList();

  } catch (error) {
    alert('儲存Todo失敗:' + error.message);
  }
}

</script>

<template>
  <header>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>串接 Todo List API</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" 
        integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
  </header>

  <div class="container">
    <div class="row justify-content-center">
      <h1 class="col-md-6 col-xl-4 mb-4 py-2 bg-warning rounded-3 text-center">串接 Todo List API</h1>
      <h1 class="text-center">{{ isRegister ? '註冊' : isLogin ? '主頁' : '登入' }}</h1>
      <div class="col-md-6">
        <div class="card mt-5">
          <div class="card-body" v-if="!isLogin">
            <form @submit.prevent="handleSubmit">
              <div>
                <label class="form-label">帳號(電子信箱)</label><br>
                <input type="email" class="form-control" placeholder="請輸入Email" v-model="email" required />
              </div>
              <div>
                <label class="form-label">密碼</label><br>
                <input type="password" class="form-control" placeholder="請輸入Password" v-model="password" required/>
              </div>
              <div v-if="isRegister">
                <label class="form-label">暱稱</label><br>
                <input type="text" class="form-control" placeholder="請輸入暱稱" v-model="nickname" required />
              </div>
              <hr>
              <div>
                <button type="submit" class="btn btn-info text-center" @click="isRegister ? signUp() : signIn()">{{ isRegister ? '註冊' : '登入' }}</button> |
                <button type="button" class="btn btn-secondary text-center" @click="switchPage">{{ isRegister ? '返回登入頁' : '我要註冊' }}</button>
              </div>
            </form>
          </div>
          <div class="card-body" v-else>
            <form @submit.prevent="handleSubmit">
              <div>
                <button type="submit" class="btn btn-primary text-center" @click="signOut">登出</button>
              </div>
              <hr>
              <div>
                <label class="form-label">驗證Token:</label><br>
                <input type="password" class="form-control" placeholder="請輸入Token" v-model="token" required />
                <button type="submit" class="btn btn-success text-center" @click="validToken">驗證Token</button>
              </div>
              <hr>
            </form>
            <!--TodoList-->
            <h2 class="text-center">TodoList</h2>
            <div class="input-group col-md-8">
              <input type="text" class="form-control" placeholder="請輸入Todo待辦事項" v-model="inputTodo" required /> 
              <button type="button" class="btn btn-link" @click="addTodo">新增</button>
            </div>
            <hr>
            <ul class="list-group">
              <li class="list-group-item list-group-item-warning d-flex justify-content-between align-items-center"
                  v-for="todo in todoList" :key="todo.id">
                <label v-show="!todo.isEdit">{{ todo.content }}</label>
                <input type="text" class="form-control" v-show="todo.isEdit" v-model="todo.content" />
                <div>
                  <button type="button" class="btn btn-link" @click="editTodo(todo)">{{ todo.isEdit ? '儲存' : '編輯' }}</button>
                  <button type="button" class="btn btn-danger" @click="delTodo(todo.id)">刪除</button>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </div>
    
      
    </div>
    <div></div>
  </div>
</template>

