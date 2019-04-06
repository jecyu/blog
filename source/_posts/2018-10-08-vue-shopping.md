---
author: jecyu
layout: post
title: "Vue 电商实战"
date: 2018-10-08
comments: true
categories: js相关
tags: [vue]
toc: true
---

## 导语

自上次的知乎日报项目后，对 vue 开发 web 应用有了一定的实践，本次的电商项目包含了 vue 路由和状态管理，算是 vue 全家桶的实践，其中更是体验到了使用 ES6 的优雅便捷。本文是对 《Vue.js 实战》电商实战的笔记整理和实现过程中的心得体会。

典型的场景：

- 商品列表按照价格、销量排序
- 商品列表按照品牌、价格过滤
- 动态的购物车
- 使用优惠码

<!-- more -->

## 需求分析

- 商品列表模块
  用于展示所有商品页面，一般具有筛选和过滤功能，后期可以添加上搜索关键功能。本项目是在前端一次性拿到所有数据在本地实现的，如果需要分页，则在服务端进行筛选和排序。
- 商品详情模块
  点击商品获取到它的 id，进入到商品详情的页面。
- 购物车模块
  可以对商品数据进行动态修改

## 数据状态管理

基于 Vue 的应用开发中，在理解好需求后，最先要准备的是数据，再到显示和操作数据。

### 商品列表页

本地采用 mock 数据，再导入到 Vuex 统一管理，最后通过计算属性来引用。注意的是：这里需要通知服务端，获取是异步的，所以使用 action 来操作数据。因为 mutations 只能同步操作。

    const state = {
      // 商品列表数据
      productList: []
    };

    const mutations = {
      // 添加商品列表
      setProductList(state, data) {
        state.productList = data;
      }
    };

    const actions = {
      // 请求商品列表
      getProductList(context) {
        // 真实环境通过 Ajax 获取，这里用异步模拟
        setTimeout(() => {
          context.commit("setProductList", product_data);
        }, 500);
      }
    };

使用 modules 管理状态时，在组件引用的时候也要做好相应的处理，加上模块的作用域

    Before:
    return this.$store.state.cartList;
    // 初始化时，通过 Vuex 的 action 请求数据
    this.$store.dispatch("getProductList");


    After:
    return this.$store.state.cart.cartList;
    // 初始化时，通过 Vuex 的 action 请求数据

this.$store.dispatch("product/getProductList");

对于 state 或 getter 的话，可以使用对应 mapState 和 mapGetters 辅助函数来引用。其中 state 相当于单个组件的 data，getter 相当于单个组件的计算属性，对 state 里的数据进一步处理。

### 商品详情页

为了使业务更好地解祸，从商品列表页跳转至详情页时，只传递一个商品 的 id，不需要其他任何数据(虽然像商品名称、价格等数据在详情页己拿到，但不传递，重新获取〉

对于商品简介组件，每个商品的选项比较多，为方便父子组件之间传递，直接在 product.vue 中设置一个 property:info 来接受一个对象格式的数据，这样扩展性较高，父级也可以直接将获取到的数据传递过来，省去了拆分的工作。

需要注意的是， 电商网站的详情页一般为自定义 的文本和图片， 商家通过富文本编辑器以可视 化的形式编辑好商品 内 容，接口返回的是 html 片段，可以直接用 v-html 指令渲染 html 内容， 但在服务端要对提交的 html 做处理， 避免发生 xss 攻击。

### 购物车

**准备数据：**

- Vuex 中的购物车数据 cartList
- product. 中所有的商品数据（ mock 用）。
- 将 product. 中的数组转换为字典 productDictList，方便快速选取
- 商品总数 countAll.．
- 总费用（不含优惠码） costAll.

修改购物车数量时，判断是否只有 1 件的逻辑是写在 handleCount 方法内的，而不是在 Vuex 的 editCarCount 中 。但事实上，写在 Vuex 里也是可以的，但不建议这样写， 因为 Vuex 主要以操作数据为主， 不应该关心具体的业务逻辑，业务逻辑应该在业务组件中维护 。

## 路由应用

    const router = new Router({
      mode: "history",
      base: process.env.BASE_URL,
      routes: [
        {
          path: "/",
          name: "list",
          meta: {
            title: "商品列表"
          },
          // 赖加载组件
          component: () =>
            import(/* webpackChunkName: "list" */ "../views/List.vue")
        },
    }

    // 更改页面 title，以及滚动定位到页面顶部
    router.beforeEach((to, from, next) => {
      window.document.title = to.meta.title;
      console.log(to);
      next();
    });

    router.afterEach(() => {
      window.scrollTo(0, 0);
    });

### 应用

路由视图 `<router-view>` 挂载了所有的路由组件

    <div class="header">
      <router-link to="/list" class="header-title">花森淘</router-link>
      <div class="header-menu"></div>
      <router-link to="/cart" class="header-menu-cart">
        购物车
        <span v-if="cartList.length"> {{ cartList.length }} </span>
      </router-link>
    </div>
    <router-view/>

## 总结

在大中型项目中，最重要的是模块解耦。对于公共组件，要定义好 API ( props、 events、 slots ），公用数据要在 Vuex 或 bus 中统一维护。 在业务中， 要尽可能避免直接操作父链和子链来修改组件的状态， 对于跨级通信最好通过 Vuex 或 bus 完成。

在协同开发时，可以将路由组件的内容拆分为多个组件，由不同的人维护， 这样可以避免冲 突，使模块更清晰， 寻找 bug 也更有针对性。 公共配置还可以使用混合（ mixins ）

当项目中页面较多，在使用 Vuex 时可以将 store 分发到不同的文件或文件夹内，使用 modules 把 store 分割到不同的模板， 这样对于复杂的应用更具维护性。

> 本项目 github 地址：https://github.com/Jecyu/shopping

（完）
