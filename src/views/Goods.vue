<template>
  <div class="container">
    <section>
      <div v-if="!customer" class="login" @click="toLogin">
        <p>登录后享受更多优惠</p>
        <p>去登录 <i class="fa fa-angle-right"></i></p>
      </div>
      <img :src="goods.goodsImg" />
      <div class="price">
        <p><span>￥</span>{{ goods.goodsPrice }}</p>
      </div>
      <div class="info">
        <h3>{{ goods.goodsName }}</h3>
        <p>{{ goods.goodsExplain }}</p>
      </div>
      <div style="width: 100%; height: 14.4vw"></div>
    </section>
    <footer>
      <ul>
        <li @click="toIndex">
          <i class="fa fa-home"></i>
          <p>首页</p>
        </li>
        <li>
          <i class="fa fa-volume-control-phone"></i>
          <p>客服</p>
        </li>
        <li @click="toCart">
          <i class="fa fa-shopping-cart"></i>
          <p>购物车</p>
          <div class="quantity" v-if="cartcount > 0">{{ cartcount }}</div>
        </li>
      </ul>
      <div class="cart">
        <div class="cart-box" @click="addCart">加入购物车</div>
      </div>
    </footer>
  </div>
</template>
<script setup>
import { useRouter, useRoute } from "vue-router";
import { ref, inject } from "vue";
import { getSessionStorage } from "../common.js";
import { onMounted } from "vue";

const axios = inject("axios");
const router = useRouter();
const route = useRoute();
const goods = ref({});
const cartcount = ref(0);
const loginElement = ref(null);
const firstP = ref(null);
const secondP = ref(null);
const customer = getSessionStorage("customer");
//查询购物中是否存在商品
let selectCartCountByTelId = () => {
  axios
    .get("selectCartCountByTelId", {
      params: {
        telId: customer.telId,
      },
    })
    .then((response) => {
      cartcount.value = response.data;
    })
    .catch((error) => {
      console.log(error);
    });
};

// 获取商品详情并合并到购物车数据中
const fetchProductDetails = (cartItem) => {
  return axios
    .get("selectGoodsById", { params: { goodsId: cartItem.goodsId } }) // 调用 getByGoodId 接口
    .then((response) => {
      if (response.data && response.data.length > 0) {
        const goods = response.data[0]; // 获取返回的商品数据
        // 将商品的详细信息（名称、价格、图片等）合并到购物车项
        cartItem.goodsName = goods.goodsName;
        cartItem.goodsImg = goods.goodsImg;
        cartItem.goodsPrice = goods.goodsPrice;
      } else {
        console.error(`商品ID为${cartItem.goodsId}的商品信息未找到`);
      }
    })
    .catch((error) => {
      console.error("获取商品详情失败：", error);
    });
};

//初始化
const init = () => {
  axios
    .get("selectGoodsById", {
      params: {
        goodsId: String(route.query.goodsId),
      },
    })
    .then((response) => {
      // 检查后端返回的数据（它是一个数组）
      console.log("商品详情返回的数据：", response.data);

      // 将商品数组中的第一个元素赋值给 goods
      goods.value = response.data[0]; // 获取数组中的第一个商品

      console.log("加载的商品数据：", goods.value); // 打印加载的商品数据
    })
    .catch((error) => {
      console.log("获取商品详情失败：", error);
    });

  if (customer != null) {
    selectCartCountByTelId();
  }
};

init();
onMounted(() => {
  // 判断用户是否登录
  const customer = getSessionStorage("customer");
  if (customer) {
    console.log("用户已登录");
    // 如果用户已登录，禁用登录区域的点击事件并更新文字内容
    const loginElement = document.querySelector(".login");
    const pElements = document.querySelectorAll(".login p");

    if (loginElement && pElements.length > 0) {
      pElements[0].innerText = " ";
      pElements[1].innerText = " ";
      loginElement.removeAttribute("onclick"); // 禁用点击事件
    }
  } else {
    console.log("用户未登录");
    // 如果用户未登录，保持正常状态
    const loginElement = document.querySelector(".login");
    const pElements = document.querySelectorAll(".login p");

    if (loginElement == null) console.log("loginElement null");
    if (pElements.length == 0) console.log("pElements null");

    if (loginElement && pElements.length > 0) {
      pElements[0].innerText = "登录后享受更多优惠"; // 设置第一行文字
      pElements[1].innerText = "去登录"; // 设置第二行文字
      pElements[1].innerHTML += ' <i class="fa fa-angle-right"></i>'; // 添加右箭头
      loginElement.setAttribute("onclick", "toLogin()"); // 恢复点击事件
    }
  }
});

const updateQuantityCart = () => {
  axios
    .put("updateQuantityCart", {
      telId: customer.telId,
      goodsId: route.query.goodsId,
      quantity: 1,
    })
    .then((response) => {
      if (response.data.affectedRows == 1) {
        alert("加入购物车成功！");
      } else {
        alert("加入购物车失败！");
      }
    })
    .catch((error) => {
      console.log(error);
    });
};

const insertCart = () => {
  axios
    .post("insertCart", {
      telId: customer.telId,
      goodsId: route.query.goodsId,
      quantity: 1,
      state: 1,
    })
    .then((response) => {
      if (response.data.affectedRows == 1) {
        alert("加入购物车成功！");
        selectCartCountByTelId();
      } else {
        alert("加入购物车失败！");
      }
    })
    .catch((error) => {
      console.log(error);
    });
};

//添加购物车
const addCart = () => {
  if (customer == null) {
    router.push("/login");
  } else {
    console.log("顾客电话：" + customer.telId);
    axios
      .post("insertCart", {
        telId: customer.telId,
        goodsId: route.query.goodsId,
        quantity: 1,
      })
      .then((response) => {
        // 进行查询，如果购物车中已有该物品，修改商品数量
        //如果没有，则进行插入
        if (response.data) {
          // updateQuantityCart(); //修改商品数量
          alert("加入购物车成功！");
        } else {
          // insertCart(); //插入新的商品
          // console.log("准备开始插入新的商品");
          alert("加入购物车失败！");
        }
      })
      .catch((error) => {
        console.log(error);
      });
  }
};
const toCart = () => {
  router.push("/cart");
};
const toLogin = () => {
  router.push("/login");
};
const toIndex = () => {
  router.push("/index");
};
</script>
<style scoped>
/*********************** 总容器 *********************/
.container {
  width: 100%;
  height: 100%;
}
/*********************** section 部分 *********************/
section {
  width: 100%;
}
section .login {
  width: 100%;
  height: 10vw;
  box-sizing: border-box;
  padding: 0 4.4vw;
  font-size: 4.2vw;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
section .login p:last-child {
  font-size: 3.6vw;
  color: #666;
  user-select: none;
  cursor: pointer;
}
section .login p:last-child i {
  font-size: 4.6vw;
  margin-left: 2vw;
}
section img {
  width: 100%;
  height: 100vw;
  box-shadow: 0 2px 4px -1px #e5e5e5;
}
section .price {
  width: 100%;
  height: 14vw;
  box-sizing: border-box;
  padding-left: 4.4vw;
  display: flex;
  align-items: center;
  font-size: 6.4vw;
  font-weight: 700;
  color: #ff6700;
}
section .price p {
  font-size: 6.4vw;
  font-weight: 700;
  color: #ff6700;
}
section .price p span {
  font-size: 3.2vw;
  vertical-align: text-top;
}
section .info {
  width: 100%;
  box-sizing: border-box;
  padding-left: 4.4vw;
}
section .info h3 {
  font-size: 4.4vw;
  margin-bottom: 3vw;
}
section .info p {
  font-size: 3.3vw;
}
/*********************** footer 部分 *********************/
footer {
  width: 100%;
  height: 14.4vw;
  background-color: #fff;
  box-shadow: 0 3px 14px 2px #ccc;
  display: flex;
  align-items: center;
  position: fixed;
  left: 0;
  bottom: 0;
}
footer ul {
  flex: 2;
  display: flex;
  justify-content: space-around;
  align-items: center;
  font-size: 3.3vw;
  color: #999;
}
footer ul li {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  user-select: none;
  cursor: pointer;
  position: relative;
}
footer ul li .fa {
  font-size: 5vw;
  color: #666;
  margin-bottom: 1.2vw;
}
footer ul li:last-child .quantity {
  width: 4.4vw;
  height: 4.4vw;
  background-color: #ff6700;
  text-align: center;
  line-height: 4.4vw;
  color: #fff;
  font-size: 3.3vw;
  border-radius: 2.2vw;
  position: absolute;
  left: 7vw;
  top: -1.2vw;
}
footer .cart {
  flex: 3;
  display: flex;
  justify-content: center;
  align-items: center;
}
footer .cart .cart-box {
  width: 80%;
  height: 10vw;
  line-height: 10vw;
  text-align: center;
  background-image: linear-gradient(to right, #ff9e22, #ff5934);
  border-radius: 5vw;
  font-size: 4vw;
  color: #fff;
  user-select: none;
  cursor: pointer;
}
</style>
