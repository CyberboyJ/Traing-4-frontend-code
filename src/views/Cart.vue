<template>
  <div class="container">
    <header>
      <p>购物车</p>
    </header>
    <section>
      <div style="width: 100%; height: 13.9vw"></div>
      <img
        class="cart-null"
        v-if="cartArr.length == 0"
        src="../assets/cartnull.png"
      />
      <ul>
        <li v-for="cart in cartArr" :key="cart.cartId">
          <div>
            <i
              class="fa fa-check-circle"
              v-show="cart.state == 1"
              @click="isSelect(cart.goodsId, cart.state)"
            ></i>
            <i
              class="fa fa-circle-o"
              v-show="cart.state == 0"
              @click="isSelect(cart.goodsId, cart.state)"
            ></i>
          </div>
          <div>
            <img :src="cart.goodsImg" />
          </div>
          <div>
            <h3>{{ cart.goodsName }}</h3>
            <p>售价：{{ cart.goodsPrice }}元</p>
            <div class="quantity-box">
              <div class="quantity-box-left">
                <div
                  class="change"
                  @click="changeQuantity(cart.goodsId, cart.quantity, -1)"
                >
                  -
                </div>
                <div class="quantity">{{ cart.quantity }}</div>
                <div
                  class="change"
                  @click="changeQuantity(cart.goodsId, cart.quantity, 1)"
                >
                  +
                </div>
              </div>
              <div class="quantity-box-right">
                <i class="fa fa-trash-o" @click="delGoods(cart.goodsId)"></i>
              </div>
            </div>
          </div>
        </li>
      </ul>
      <div style="width: 100%; height: 14.4vw"></div>
    </section>
    <footer>
      <ul>
        <li>
          <p>共{{ goodsTotalQuantity }}件</p>
          <p>
            金额：<span>{{ goodsTotalPrice }}</span
            >元
          </p>
        </li>
        <li @click="toGoodsType">继续购物</li>
        <li v-show="goodsTotalQuantity >= 1" @click="toSettlement">去结算</li>
      </ul>
    </footer>
  </div>
</template>
<script setup>
import { useRouter } from "vue-router";
import { ref, inject, computed } from "vue";
import { getSessionStorage } from "../common.js";
const axios = inject("axios");
const router = useRouter();
const cartArr = ref([]);
const customer = getSessionStorage("customer");
//使用计算属性获取商品总数量和总金额
const goodsTotalQuantity = computed(() => {
  let totalQuantity = 0;
  for (let i = 0; i < cartArr.value.length; i++) {
    if (cartArr.value[i].state == 1) {
      totalQuantity += cartArr.value[i].quantity;
    }
  }
  return totalQuantity;
});
const goodsTotalPrice = computed(() => {
  let totalPrice = 0;
  for (let i = 0; i < cartArr.value.length; i++) {
    if (cartArr.value[i].state == 1) {
      totalPrice += cartArr.value[i].quantity * cartArr.value[i].goodsPrice;
    }
  }
  return totalPrice;
});
//初始化
// 获取商品详情并合并到购物车数据中
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

// 初始化购物车
const init = () => {
  if (!customer || !customer.telId) {
    console.error("未找到客户的手机号码（telId）。");
    return;
  }

  console.log("开始加载购物车数据...");

  axios
    .get(`selectCartByTelId/${customer.telId}`) // 使用路径参数
    .then((response) => {
      cartArr.value = response.data; // 保存购物车数据
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
// 改变购物车中商品被勾选的状态
const isSelect = (goodsId, state) => {
  // 确保 goodsId 为字符串类型
  const goodsIdStr = String(goodsId); // 强制转换为字符串

  console.log("发送请求的数据:", {
    telId: customer.telId,
    goodsId: goodsIdStr, // 传递字符串类型的 goodsId
    state: state == 1 ? 0 : 1,
  });

  axios
    .put("updateCartState", {
      telId: customer.telId,
      goodsId: goodsIdStr, // 确保传递给后端的是字符串
      state: state == 1 ? 0 : 1,
    })
    .then((response) => {
      console.log("更新商品状态响应：", response.data);
      if (response.data) {
        init(); // 重新加载购物车数据
      } else {
        alert("选择商品失败！");
      }
    })
    .catch((error) => {
      console.log(error);
      alert("选择商品请求出错！");
    });
};

// 修改商品数量
const changeQuantity = (goodsId, currentQuantity, num) => {
  // 如果商品数量小于等于1，且减少操作，则不进行处理
  if (currentQuantity <= 1 && num === -1) {
    return;
  }

  // 发送给后端的数量变更值（+1 或 -1）
  const quantityChange = num; // 正常情况下直接传递 +1 或 -1

  // 使用 PUT 方法传递数量变化给后端
  axios
    .put("updateQuantityCart", {
      telId: customer.telId,
      goodsId: String(goodsId), // 确保 goodsId 是字符串
      quantity: quantityChange, // 传递数量变化值
    })
    .then((response) => {
      console.log("获取到的商品信息~", response.data);

      // 判断后端返回的是否为 true，表示操作成功
      if (response.data === true) {
        // 找到当前更新的商品
        const updatedCartArr = [...cartArr.value]; // 创建购物车数据副本

        // 找到需要更新数量的商品
        const updatedItem = updatedCartArr.find(
          (item) => item.goodsId === goodsId
        );
        if (updatedItem) {
          updatedItem.quantity += quantityChange; // 更新商品数量
        }

        // 更新 cartArr
        cartArr.value = updatedCartArr;

        console.log("购物车数据已更新：", cartArr.value);
      } else {
        alert("更新数量失败！");
      }
    })
    .catch((error) => {
      console.log("更新数量请求出错：", error);
      alert("更新数量请求出错！");
    });
};

// 删除指定商品
const delGoods = (goodsId) => {
  if (!confirm("确认要删除此商品吗？")) {
    return;
  }

  // 使用 DELETE 方法，传递 telId 和 goodsId
  axios
    .delete("deleteCartByTeIdByGoodsId", {
      params: {
        // 使用 params 而不是 body 传递查询参数
        telId: customer.telId,
        goodsId: String(goodsId), // 确保 goodsId 是字符串
      },
    })
    .then((response) => {
      console.log("删除商品响应：", response.data);

      // 检查后端返回的 response.data 是否为 true，表示删除成功
      if (response.data === true) {
        init(); // 删除成功后重新加载购物车数据
      } else {
        alert("删除商品失败！");
      }
    })
    .catch((error) => {
      console.log("删除商品请求出错：", error);
      alert("删除商品请求出错！");
    });
};

const toGoodsType = () => {
  router.push("/goodsType");
};
const toSettlement = () => {
  router.push("/settlement");
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
  background-color: #eee;
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
section .cart-null {
  width: 100vw;
}
section ul {
  /*display:none;*/
  width: 100%;
}
section ul li {
  width: 100%;
  height: 28vw;
  box-sizing: border-box;
  padding: 3vw;
  background-color: #fff;
  margin-bottom: 3.6vw;
  display: flex;
}
section ul li > div:nth-child(1) {
  flex: 0 0 10vw;
  display: flex;
  justify-content: center;
  align-items: center;
}
section ul li > div:nth-child(2) {
  flex: 0 0 26vw;
}
section ul li > div:nth-child(3) {
  flex: 1;
  box-sizing: border-box;
  padding-left: 3vw;
}
section ul li > div:nth-child(1) i {
  font-size: 6vw;
  color: #ff6700;
}
section ul li > div:nth-child(2) img {
  width: 26vw;
  height: 22vw;
  box-sizing: border-box;
  border: solid 1px #ddd;
}
section ul li > div:nth-child(3) h3 {
  font-weight: 300;
  font-size: 4vw;
  color: #666;
  margin-bottom: 1vw;
}
section ul li > div:nth-child(3) p {
  font-size: 3.3vw;
  color: #999;
  margin-bottom: 1vw;
}
section ul li > div:nth-child(3) .quantity-box {
  width: 100%;
  display: flex;
  justify-content: space-between;
}
section ul li > div:nth-child(3) .quantity-box .quantity-box-left {
  width: 24vw;
  height: 9vw;
  border: solid 1px #eee;
  display: flex;
}
section ul li > div:nth-child(3) .quantity-box .quantity-box-left .change {
  width: 9vw;
  height: 9vw;
  background-color: #f4f4f4;
  color: #666;
  font-size: 6vw;
  line-height: 9vw;
  text-align: center;
  user-select: none;
  cursor: pointer;
}
section ul li > div:nth-child(3) .quantity-box .quantity-box-left .quantity {
  width: 9vw;
  height: 9vw;
  color: #666;
  font-size: 5vw;
  line-height: 9vw;
  text-align: center;
  user-select: none;
  cursor: pointer;
}
section ul li > div:nth-child(3) .quantity-box .quantity-box-right {
  height: 9vw;
  line-height: 9vw;
}
section ul li > div:nth-child(3) .quantity-box .quantity-box-right i {
  color: #ccc;
  font-size: 6vw;
  margin-right: 2vw;
  user-select: none;
  cursor: pointer;
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
  font-size: 3.9vw;
  user-select: none;
  cursor: pointer;
}
footer ul li:nth-child(1) {
  flex-direction: column;
  font-size: 3.6vw;
  color: #999;
}
footer ul li:nth-child(1) p span {
  font-weight: 700;
  font-size: 5vw;
  color: #ff6700;
}
footer ul li:nth-child(2) {
  background-color: #f4f4f4;
  color: #333;
}
footer ul li:nth-child(3) {
  background-color: #ff6700;
  color: #fff;
}
</style>