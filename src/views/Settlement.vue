<template>
  <div class="container">
    <header>
      <p>用户结算</p>
    </header>
    <section>
      <div style="width: 100%; height: 13.9vw"></div>
      <div class="address">
        <ul>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
          <li></li>
        </ul>
        <div class="content">
          <div class="content-left">
            <h3>{{ address.contactName }} {{ address.contactTel }}</h3>
            <p>{{ address.address }}</p>
          </div>
          <div class="content-right" onclick="location.href='addressList.html'">
            <i class="fa fa-angle-right"></i>
          </div>
        </div>
      </div>
      <div class="detailed">
        <ul>
          <li v-for="cart in cartArr" :key="cart.cartId">
            <div class="detailed-left">
              <img :src="cart.goodsImg" />
              <p>{{ cart.goodsName }}</p>
            </div>
            <div class="detailed-right">
              <p>￥{{ cart.goodsPrice }}</p>
              <p>x {{ cart.quantity }}</p>
            </div>
          </li>
        </ul>
      </div>
      <div class="payment">
        <ul>
          <li>
            <div class="payment-left">
              <img src="../assets/payment01.png" />
              <p>支付宝</p>
            </div>
            <input
              type="radio"
              id="alipay"
              v-model="selectedPaymentMethod"
              value="alipay"
              name="payment"
            />
          </li>
          <li>
            <div class="payment-left">
              <img src="../assets/payment02.png" />
              <p>小米钱包</p>
            </div>
            <input
              type="radio"
              id="xiaomi"
              v-model="selectedPaymentMethod"
              value="xiaomi"
              name="payment"
            />
          </li>
          <li>
            <div class="payment-left">
              <img src="../assets/payment03.png" />
              <p>微信支付</p>
            </div>
            <input
              type="radio"
              id="wechat"
              v-model="selectedPaymentMethod"
              value="wechat"
              name="payment"
            />
          </li>
        </ul>
      </div>
      <div style="width: 100%; height: 14.4vw"></div>
    </section>
    <footer>
      <ul>
        <li>共{{ goodsTotalQuantity }}件 合计：{{ goodsTotalPrice }}</li>
        <li @click="toPayment">去付款</li>
      </ul>
    </footer>
  </div>
</template>
<script setup>
import { useRouter } from "vue-router";
import { ref, inject, computed } from "vue";
import { nextTick } from "vue"; //额外添加的，用于address（1.4）
import { getSessionStorage, getCurDate, getCurTime } from "../common.js";
const axios = inject("axios");
const router = useRouter();
const address = ref({});
const cartArr = ref([]);
const customer = getSessionStorage("customer");
//支付方式默认选择支付宝
const selectedPaymentMethod = ref("alipay"); // 默认选择支付宝
//使用计算属性获取商品总数量和总金额
const goodsTotalQuantity = computed(() => {
  let totalQuantity = 0;
  for (let i = 0; i < cartArr.value.length; i++) {
    totalQuantity += cartArr.value[i].quantity;
  }
  return totalQuantity;
});
const goodsTotalPrice = computed(() => {
  let totalPrice = 0;
  for (let i = 0; i < cartArr.value.length; i++) {
    totalPrice += cartArr.value[i].quantity * cartArr.value[i].goodsPrice;
  }
  return totalPrice;
});

// 获取商品详情并合并到购物车数据中（Cart.vue里面也有这个）
const fetchProductDetails = (cartItem) => {
  return new Promise((resolve, reject) => {
    axios
      .get("selectGoodsById", { params: { goodsId: cartItem.goodsId } })
      .then((response) => {
        if (response.data && response.data.length > 0) {
          const goods = response.data[0]; // 获取返回的商品数据
          // 将商品的详细信息（名称、价格、图片等）合并到购物车项
          cartItem.goodsName = goods.goodsName;
          cartItem.goodsImg = goods.goodsImg;
          cartItem.goodsPrice = goods.goodsPrice;
          resolve(cartItem); // 返回已更新的 cartItem
        } else {
          console.error(`商品ID为${cartItem.goodsId}的商品信息未找到`);
          reject(`商品ID为${cartItem.goodsId}的商品信息未找到`);
        }
      })
      .catch((error) => {
        console.error("获取商品详情失败：", error);
        reject(error); // 失败时返回错误
      });
  });
};

//初始化
const init = () => {
  const defaultValue = customer.default || 1; // 确保 default 参数传递
  axios
    .get("address", {
      params: {
        telId: customer.telId,
        default: String(defaultValue), // 确保添加 default 参数，表示查询默认地址[指导书上要求的参数包含这个，所以为前端代码额外添加]（1.4）
      },
    })
    .then((response) => {
      console.log("返回的地址数据：", response.data); // 确保输出返回的数据
      address.value = response.data[0];
      console.log("返回的地址数据2：", address.value);
    })
    .catch((error) => {
      console.log(error);
    });
  //再次获取购物车信息
  axios
    .get(`selectCartByTelId/${customer.telId}`) // 使用路径参数
    .then((response) => {
      // cartArr.value = response.data; // 保存购物车数据
      cartArr.value = response.data.filter((item) => item.state === 1); // 筛选已勾选的商品
      console.log("购物车数据：", cartArr.value);

      // 对每个购物车商品获取详细信息
      const productDetailsPromises = cartArr.value.map((cartItem) =>
        fetchProductDetails(cartItem)
      );

      // 等待所有商品的详细信息都加载完成
      Promise.all(productDetailsPromises)
        .then(() => {
          console.log("所有商品的详细信息已加载：", cartArr.value);
        })
        .catch((error) => {
          console.error("加载商品信息时出错：", error);
        });
    })
    .catch((error) => {
      console.log("获取购物车数据失败：", error);
    });
};
init();
// 创建订单
// 创建订单
const toPayment = () => {
  // 构建订单详细信息（orderDetails），确保它是一个数组
  const orderDetails = cartArr.value.map((cartItem) => ({
    goodsId: String(cartItem.goodsId), // 确保goodsId为字符串
    quantity: String(cartItem.quantity), // 确保quantity为字符串
  }));

  // 确保 orderDetails 是一个非空数组
  if (orderDetails.length === 0) {
    alert("订单详情不能为空！");
    return;
  }

  // 确保 addressId 和 orderTotal 是字符串
  const addressId = String(address.value.addressId);
  const orderTotal = String(goodsTotalPrice.value);

  // 创建订单请求体
  const orderData = {
    telId: customer.telId, // 用户电话
    addressId: addressId, // 地址ID（确保为字符串）
    orderTotal: orderTotal, // 总金额（确保为字符串）
    orderDetails: orderDetails, // 商品详情（数组，传递给后端）
  };

  // 发送POST请求到后端创建订单
  axios
    .post("createOrder", orderData)
    .then((response) => {
      console.log("订单创建结果", response.data);

      if (response.data.success) {
        // 跳转到支付页面
        router.push({
          path: "/payment",
          query: { orderId: response.data.orderId },
        });
      } else {
        alert("生成订单失败！");
      }
    })
    .catch((error) => {
      console.log("请求失败：", error); // 错误处理
      alert("订单创建失败，请稍后重试！");
    });
};
</script>
<style scoped>
/*********************** 总容器 *********************/
.container {
  width: 100%;
  height: 100%;
  background-color: #eee;
}
/*********************** header 头部 *********************/
header {
  width: 100%;
  height: 13.9vw;
  position: fixed;
  left: 0;
  top: 0;
  z-index: 100;
  /*box-shadow: 0 2px 4px -1px #CCC;*/
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
section .address {
  width: 100%;
  background-color: #fff;
  border-top: solid 1px #ddd;
  border-bottom: solid 1px #ddd;
  margin-bottom: 1.8vw;
}
section .address ul {
  width: 100%;
  height: 1vw;
  display: flex;
  justify-content: space-between;
}
section .address ul li {
  flex: 1;
  height: 1vw;
  margin: 0 0.6vw;
  transform: skew(-30deg);
}
section .address ul li:nth-child(odd) {
  background-color: #ff8a81;
}
section .address ul li:nth-child(even) {
  background-color: #8fcaf9;
}
section .address .content {
  width: 100%;
  box-sizing: border-box;
  padding: 5.6vw;
  display: flex;
  justify-content: space-between;
}
section .address .content .content-left {
  flex: 0 0 80vw;
}
section .address .content .content-left h3 {
  font-size: 4.2vw;
  color: #3c3c3c;
  margin-bottom: 2vw;
}
section .address .content .content-left p {
  font-size: 3.3vw;
  color: #757575;
}
section .address .content .content-right {
  flex: 1;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  user-select: none;
  cursor: pointer;
}
section .address .content .content-right i {
  font-size: 6vw;
  color: #3c3c3c;
}
section .detailed {
  width: 100%;
  margin-bottom: 1.8vw;
  background-color: #fff;
  border-top: solid 1px #ddd;
  border-bottom: solid 1px #ddd;
}
section .detailed .info {
  width: 100%;
  box-sizing: border-box;
  padding: 3.6vw 5vw;
  font-size: 3.3vw;
  color: #3c3c3c;
}
section .detailed ul {
  width: 100%;
}
section .detailed ul li {
  width: 100%;
  height: 18vw;
  box-sizing: border-box;
  padding: 2vw 5vw;
  border-top: solid 1px #eee;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 3.3vw;
  color: #3c3c3c;
}
section .detailed ul li .detailed-left {
  display: flex;
  align-items: center;
}
section .detailed ul li .detailed-left img {
  width: 16vw;
  height: 14vw;
  margin-right: 2vw;
}
section .detailed ul li .detailed-right {
  text-align: right;
}
section .detailed ul li .detailed-right p:last-child {
  margin-top: 2vw;
  color: #bbb;
}
section .payment {
  width: 100%;
  border-top: solid 1px #ddd;
  border-bottom: solid 1px #ddd;
  background-color: #fff;
}
section .payment ul {
  width: 100%;
}
section .payment ul li {
  width: 89vw;
  height: 12.5vw;
  margin: 0 auto;
  border-bottom: solid 1px #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
section .payment ul li .payment-left {
  display: flex;
  align-items: center;
}
section .payment ul li .payment-left img {
  width: 8.9vw;
  height: 8.9vw;
  margin-right: 3vw;
}
section .payment ul li .payment-left p {
  font-size: 3.6vw;
  color: #333;
}
section .payment ul li input {
  width: 4.5vw;
  height: 4.5vw;
}
/*********************** footer 部分 *********************/
footer {
  width: 100%;
  height: 14.4vw;
  background-color: #fff;
  box-shadow: 0 3px 14px 2px #ccc;
  position: fixed;
  left: 0;
  bottom: 0;
  display: flex;
}
footer ul {
  width: 100%;
  display: flex;
}
footer ul li {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 4.2vw;
  color: #ff6700;
  user-select: none;
  cursor: pointer;
}
footer ul li:last-child {
  background-color: #ff6700;
  color: #fff;
}
</style>
