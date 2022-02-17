<script setup>
import { reactive, ref } from "@vue/reactivity";
import { ElInput } from "element-plus";
import axios from "axios";
import router from "@/router";
import users from "@/assets/users.json";


const form = reactive({
  name:"",
  pass:""
})

const onSubmit = async (e) => {
  if (users.find((el)=>el.login==form.name&&el.pass==form.pass)) {
    localStorage.setItem("token","token")
    router.push("/")
  } else {
    alert("Invalid login or password")
  }
}

</script>

<template>
  <div class="login-wrapper">
    <div class="login">
      <img src="@/assets/logo.png" alt="">
      <span class="title">Вход</span>
      <ElForm v-model="form" ref="formRef" label-width="70px" label-position="top">
        <ElFormItem label="Логин">
          <ElInput v-model="form.name" placeholder="" />
        </ElFormItem>
        <ElFormItem label="Пароль">
          <ElInput type="password" v-model="form.pass" show-password/>
        </ElFormItem>
        <ElFormItem>
        <el-button type="primary" @click="onSubmit">Submit</el-button>
        </ElFormItem>
      </ElForm>
    </div>
  </div>
</template>

<style>
.login-wrapper {
  display: flex;
  place-items: center;
  height: 100vh;
}

.login {
  padding: 100px;
  border:1px solid var(--color-border);
  border-radius: 5px;
  display: flex;
  flex-direction: column;
  place-items: center;
  margin: 0 auto;
}
.login > * {
  margin: 5px;
}
.login .title {
  font-size: 18px;
  font-weight: 500;
}
.login img {
  width: 88px;
  height: 88px;
}
.login .el-button {
  margin: 0 auto;
}
@media (min-width: 1024px) {
  .login {
    min-width: 500px;
  }
}
</style>
