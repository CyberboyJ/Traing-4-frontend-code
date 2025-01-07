<template>
  <div class="container">
    <header>
      <p>支付平台页面</p>
    </header>
    <section>
      <div style="width: 100%; height: 13.9vw"></div>
      <div class="payment-info">第三方支付平台页面</div>
      <div class="order-info">
        <p>订单编号：{{ orders.orderId }}</p>
        <p>订单金额：{{ orders.orderTotal }}￥</p>
        <p>客户编号：{{ orders.telId }}</p>
        <p>客户名称：{{ orders.customerName }}</p>
        <p>订单状态：等待付款</p>
        <p>收货地址：{{ orders.address }}</p>
        <p>联 系 人：{{ orders.contactName }}</p>
        <p>联系电话：{{ orders.contactTel }}</p>
        <p>订购商品：</p>
        <p v-for="(od, index) in orderdetailsArr" :key="od.odId">
          {{ index + 1 }}. {{ od.goodsName }} x {{ od.quantity }}
        </p>
      </div>
      <div style="width: 100%; height: 14.4vw"></div>
    </section>
    <Footer></Footer>
  </div>
</template>
<script setup>
import Footer from "@/components/Footer.vue";
import { useRoute } from "vue-router";
import { ref, inject } from "vue";
const axios = inject("axios");
const route = useRoute();
const orders = ref({});
const orderdetailsArr = ref([]);
//查找订单
const init = () => {
  const orderId = route.query.orderId; // 从路由中获取 orderId
  console.log("要查找的订单路由：", orderId); // 确保获取到的 orderId 正确
  if (!orderId) {
    console.log("订单号不能为空");
    return;
  }

  // 请求订单信息，传递 orderId 到路径
  axios
    .get(`/orders/${orderId}`) // 将 orderId 作为路径参数传递给后端
    .then((response) => {
      console.log("后端返回的订单信息：", response.data);
      orders.value = response.data;
    })
    .catch((error) => {
      console.log("查询订单时发生错误：", error); // 错误处理
    });

  // 请求订单详情
  axios
    .get("order-details", {
      params: { orderId: orderId }, // 假设 order-details 接口需要这个参数
    })
    .then((response) => {
      // 处理响应数据
      if (response.data.code === 0) {
        // 如果返回的 code 为 0，表示查询成功
        console.log(response.data);
        orderdetailsArr.value = response.data.data; // 这里是返回的订单详情数据
        console.log("查询到的订单详情：", orderdetailsArr.value);
      } else {
        // 如果返回的 code 不是 0，表示查询失败
        console.error("查询订单详情失败：", response.data.message);
      }
    })
    .catch((error) => {
      // 错误处理
      console.error("查询订单详情时发生错误：", error);
    });

  // // 如果需要，可以继续请求订单详情
  // axios
  //   .get(`/order-details`, {
  //     params: { orderId: orderId }, // 假设 order-details 接口需要这个参数
  //   })
  //   .then((response) => {
  //     orderdetailsArr.value = response.data;
  //   })
  //   .catch((error) => {
  //     console.error("查询订单详情时发生错误：", error);
  //   });
};

init();
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
  height: 13.9vw;
  background-color: #eee;
  position: fixed;
  left: 0;
  top: 0;
  z-index: 100;
}
header p {
  line-height: 13.9vw;
  text-align: center;
  font-size: 4.2vw;
  color: #666;
}
/*********************** section 部分 *********************/
section {
  width: 100%;
}
section .payment-info {
  width: 80vw;
  margin: 0 auto;
  font-size: 5vw;
  line-height: 30vw;
  text-align: center;
}
section .order-info {
  width: 80vw;
  margin: 0 auto;
  font-size: 3.6vw;
  color: #333;
}
section .order-info p {
  margin-bottom: 1vw;
}
</style>
