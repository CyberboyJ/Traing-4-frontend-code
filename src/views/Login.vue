<template>
  <div class="container">
    <header>
      <img src="../assets/logo.png" />
      <p>让每个人都能享受科技的乐趣</p>
    </header>
    <section>
      <h1>小米账号登录</h1>
      <div class="box">
        <input
          type="text"
          v-model="coutomer.telId"
          placeholder="请输入手机号"
        />
      </div>
      <div class="box">
        <input
          type="password"
          v-model="coutomer.password"
          placeholder="请输入密码"
        />
      </div>
      <div class="btn" @click="login">登 录</div>
      <div class="btn register" onclick="location.href='register.html'">
        注 册
      </div>
      <div style="width: 100%; height: 14.4vw"></div>
    </section>
    <Footer></Footer>
  </div>
</template>
<script setup>
import Footer from "@/components/Footer.vue";
import { useRouter } from "vue-router";
import { ref, inject } from "vue";
import { setSessionStorage } from "../common.js";
const axios = inject("axios");
const router = useRouter();
const coutomer = ref({
  telId: "",
  password: "",
});
const login = () => {
  if (coutomer.value.telId == "") {
    alert("手机号码不能为空！");
    return;
  }
  if (coutomer.value.password == "") {
    alert("密码不能为空！");
    return;
  }
  console.log("手机号：" + coutomer.value.telId);
  console.log("密码：" + coutomer.value.password);
  axios
    .get("selectCustByTelAndPwd", {
      params: { tel: coutomer.value.telId, pwd: coutomer.value.password },
    })
    .then((response) => {
      console.log("查找到的用户数据：");
      console.log(response.data);
      if (response.data.length == 0) {
        alert("手机号码或密码输入错误！");
      } else {
        setSessionStorage("customer", response.data[0]);
        router.go(-1);
      }
    })
    .catch((error) => {
      console.log(error);
    });
};
//   axios
//     .post("selectCustomerByTelIdByPass", {
//       telId: coutomer.value.telId,
//       pwd: coutomer.value.password, // 改为 pwd
//     })
//     .then((response) => {
//       if (response.data.length == 0) {
//         alert("手机号码或密码输入错误！");
//       } else {
//         setSessionStorage("customer", response.data[0]);
//         router.go(-1);
//       }
//     })
//     .catch((error) => {
//       console.log(error);
//     });
// };
</script>
<style scoped>
/*********************** 总容器 *********************/
.container {
  width: 100%;
  height: 100%;
}
/*********************** header 头部 *********************/
header {
  width: 100%;
  height: 60vw;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
header img {
  width: 15vw;
  height: 15vw;
}
header p {
  font-size: 4vw;
  color: #999;
  margin-top: 4vw;
}
/*********************** 中间内容部分 *********************/
section {
  width: 100%;
  box-sizing: border-box;
  padding: 0 10vw;
}
section h1 {
  font-size: 6vw;
  margin-bottom: 5vw;
}
section div {
  margin-bottom: 5vw;
}
section .box {
  width: 100%;
  height: 12vw;
  background-color: #f0f0f0;
  border-radius: 4vw;
  display: flex;
  align-items: center;
}
section .box input {
  width: 100%;
  box-sizing: border-box;
  padding-left: 4vw;
  border: none;
  outline: none;
  background-color: #f0f0f0;
}
section .btn {
  width: 100%;
  height: 12vw;
  border-radius: 6vw;
  background-color: #62bfff;
  font-size: 4.6vw;
  color: #fff;
  line-height: 12vw;
  text-align: center;
  user-select: none;
  cursor: pointer;
}
section .register {
  background-color: #89e45c;
}
</style>