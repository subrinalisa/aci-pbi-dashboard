<template>
  <div class="login-section">
    <div class="screen">
      <div class="screen__content">
        <form class="login" @submit.prevent="handleLogin(data, $router)">
          <img src="@/assets/images/logo.png" class="w-[200px] block" alt="" />
          <div class="login__field">
            <i class="login__icon fas fa-user"></i>
            <input
              type="text"
              class="login__input"
              placeholder="User ID"
              required
              v-model="data.user_id"
            />
          </div>
          <div class="login__field">
            <i class="login__icon fas fa-lock"></i>
            <input
              type="password"
              class="login__input"
              placeholder="Password"
              required
              v-model="data.password"
            />
          </div>
          <button class="button login__submit">
            <span class="button__text">Log In Now</span>
            <a-spin class="ml-2" v-if="isLoading" />
          </button>
        </form>
      </div>
      <div class="screen__background">
        <span class="screen__background__shape screen__background__shape4"></span>
        <span class="screen__background__shape screen__background__shape3"></span>
        <span class="screen__background__shape screen__background__shape2"></span>
        <span class="screen__background__shape screen__background__shape1"></span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { apiBase } from "@/config";
import axios from "axios";
import { ref } from "vue";
import Cookies from "js-cookie";
import { showNotification } from "@/utilities/notification";

const data = ref({
  user_id: "",
  password: "",
});
const isLoading = ref(false);

const handleLogin = async (data, router) => {
  isLoading.value = true;

  try {
    const response = await axios.post(`${apiBase}/login`, data);
    isLoading.value = false;
    if (response?.data?.status === "success") {
      const userInfo = response?.data?.user;
      Cookies.set("token", response?.data?.token);

      const permissions = response?.data?.user?.all_permissions?.map(
        (permission) => permission.name
      );
      localStorage.setItem(
        "all_permissions",
        JSON.stringify(response?.data?.user?.all_permissions)
      );
      localStorage.setItem("permissions", JSON.stringify(permissions));
      localStorage.setItem("userInfo", JSON.stringify(userInfo));
      showNotification(
        "success",
        response?.data?.message || "Welcome back! You've successfully logged in"
      );
      router.push({ name: "home" });
    }
  } catch (error) {
    isLoading.value = false;
    Cookies.remove("token");
    if (error?.response?.data?.status == "error")
      showNotification(
        "error",
        error?.response?.data?.error || "Login failed. Please check your credentials and try again"
      );
    else showNotification("error", error?.message);
  }
};
</script>

<style>
.login-section {
  background: url(@/assets/images/login-bg.jpg) no-repeat center;
  background-size: cover;
  padding: 50px 0;
  min-height: 100vh;
}

.screen {
  background: linear-gradient(90deg, #5d54a4, #7c78b8);
  position: relative;
  height: 600px;
  width: 360px;
  box-shadow: 0px 0px 24px #5c5696;
  margin: auto;
}

.screen__content {
  z-index: 1;
  position: relative;
  height: 100%;
}

.screen__background {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 0;
  -webkit-clip-path: inset(0 0 0 0);
  clip-path: inset(0 0 0 0);
}

.screen__background__shape {
  transform: rotate(45deg);
  position: absolute;
}

.screen__background__shape1 {
  height: 520px;
  width: 520px;
  background: #fff;
  top: -50px;
  right: 120px;
  border-radius: 0 72px 0 0;
}

.screen__background__shape2 {
  height: 220px;
  width: 220px;
  background: #6c63ac;
  top: -172px;
  right: 0;
  border-radius: 32px;
}

.screen__background__shape3 {
  height: 540px;
  width: 190px;
  background: linear-gradient(270deg, #5d54a4, #6a679e);
  top: -24px;
  right: 0;
  border-radius: 32px;
}

.screen__background__shape4 {
  height: 400px;
  width: 200px;
  background: #7e7bb9;
  top: 420px;
  right: 50px;
  border-radius: 60px;
}

.login {
  width: 320px;
  padding: 30px;
  padding-top: 156px;
}

.login__field {
  padding: 20px 0px;
  position: relative;
}

.login__icon {
  position: absolute;
  top: 30px;
  color: #7875b5;
}

.login__input {
  border: none;
  border-bottom: 2px solid #d1d1d4;
  background: none;
  padding: 10px;
  padding-left: 24px;
  font-weight: 700;
  width: 75%;
  transition: 0.2s;
}

.login__input:active,
.login__input:focus,
.login__input:hover {
  outline: none;
  border-bottom-color: #6a679e;
}

.login__submit {
  background: #fff;
  font-size: 14px;
  margin-top: 30px;
  padding: 16px 20px;
  border-radius: 26px;
  border: 1px solid #d4d3e8;
  text-transform: uppercase;
  font-weight: 700;
  display: flex;
  align-items: center;
  width: 100%;
  color: #4c489d;
  box-shadow: 0px 2px 2px #5c5696;
  cursor: pointer;
  transition: 0.2s;
}

.login__submit:active,
.login__submit:focus,
.login__submit:hover {
  border-color: #6a679e;
  outline: none;
}

.button__icon {
  font-size: 24px;
  margin-left: auto;
  color: #7875b5;
}
</style>
