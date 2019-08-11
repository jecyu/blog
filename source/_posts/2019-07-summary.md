---
author: Jecyu
layout: post
title: "2019年7月份总结"
date: 2019-08-10
comments: true
# categories: 总结
tags: 总结
toc: true
---

## 导语

时间过得快，2019年过了快一半了。记录下七月份的总结。回顾下年初的目标。

<!-- more -->

## 工作

- 自然分析
  - [x] 内网部署智能分析旧界面。
  - [x] 组合发布产品
    - [x] 预览接入数据可视化分析二维表的展示。
    - [x] 预览二维表优化
    - [x] 预览全选目录错乱
  - [x] 完成智能分析指标数据的新界面（备注：经历界面推倒重来、需求不确定、多指标的分页等问题）
  - [x] 针对指标项的选择，二次封装 iview 一个全局下拉树组件，支持默认选择、多选、模糊搜索、勾选、取消勾选。
  - [x] 给 NrButton 和 NrSlide 组件写单元测试。
  - [x] 成果列表优化（添加删除权限、修复编辑表单重复名称问题、下载功能）。
- 一体化
  - [x] 导出PDF缺失字体处理
  - [x] 修复工具栏快速移动 hover 无响应问题
  - [x] CAD 加载颜色拾取添加样例颜色
  - [x] 修复红线预录切换做吧没有清除前面选择的文件。
  - [x] 修复红线西安做吧和国家2000无法加载 CAD 问题。
  - [x] i 查询创建微件错误
  - [x] 修复图片无法导出绘制标注
  - [x] 时间轴功能修复
    - [x] 导则地块打开与其他导则地块冲突
    - [x] 拖至历史年份，最新年份数据没有关闭（影像和地形图）
    - [x] 导则地块拖至历史年份，再返回最新年份，地图无数据显示
  - [x] 2000 系统环境部署（由于客户使用情况，只能周末加班更新）
- 其他
  - [x] 公司组件库基础环境搭建，把项目中的组件抽离出来。（下一步，需要搞好视觉样式统一）
  - [x] 系统学习前端自动化测试相关知识
    - [x] 测试框架 Jest、Mocha
    - [x] 代码覆盖率工具
    - [x] 断言库用法
    - [x] 持续集成

## 输入
  
- [x] 按照指定格式输出 Git 提交记录文件
  场景：由于要把测试环境更新到2000正式环境，需要把提交记录文件给实施同事更新，从6月份初至8月份的更新，一开始想着用git 的 `fork` 软件一条条去 copy，发现这样做太傻了。这种重复繁杂的东西还是留给机器做吧。用一行命令就能解决：
          解决：
          git log --oneline --after="2019-06-01" --name-status --author="Jecyu" --no-merges -->ChangeLog.txt
          更多用法见：https://git-scm.com/book/zh/v1/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2
- [x] vue  单元测试。[Vue NYC - Component Tests with Vue.js - Matt O’Connell](https://www.youtube.com/watch?v=OIpfWTThrK8)
- [x] Vue 源码学习，了解它的源码目录设计、构建过程，脑图记录在这里https://www.processon.com/view/link/5d4f9726e4b0145255b4d7df
- [x] require.context 的使用
  - 原理：webpack 编译时，会递归地解析所有模块，从入口文件开始，通过分析 require()、require.contenxt()、import 和 import() 表达式来构建所有模块依赖关系的图。通过 require.context() 函数构建自己的上下文，可以给这个函数传入三个参数：一个要搜索的目录，一个标记表示是否还搜索其子目录， 以及一个匹配文件的正则表达式。
  ```js
  require.context(directory, useSubdirectories = false, regExp = /^\.\//)
  ```
  - 应用：遍历目录加载所有图片，如前面的svg 图标 截图。Vue 全局组件的注册。
返回值：返回一个 require 函数，这个函数三个属性：resolve, keys(返回由 context 模块处理的请求组成的数组), id
- [x] vue 虚拟 dom 的理解
  -  前提：可以试着在浏览器中，把一个简单的 div 元素的属性打印出来，可以看到一个 DOM 元素时非常庞大的。
  - 原因：频繁的去做 DOM 更新，会产生一定的性能问题，那怎么办呢？
  - 解决：把浏览器里的Dom Tree 克隆一份完整的镜像到内存，也就是所有的“virtual DOM”，当页面的 state 发生变化以后，根据最新的 state 重新生成一份 virual DOM（相当于在内存里“刷新”整个页面），将它和之前的 virtual DOM. 做比对（diff），然后在浏览器里只渲染被改变的那部分内容，这样就解决了 浏览器的性能问题和用户体验。
  ue 中的虚拟 Dom 就是用一个原生的 JS 对象去描述一个 DOM 节点的，无论是写 template 还是通过直接写 render 函数，最后都是转换为 render 去编译的。
  render 将 dom 抽象成一个 JS 对象DOM树，以 vnode 模拟真实节点，对它进行增删改查都不需要操作真实 dom，大大提升了性能，最后通过 diff 算法得出不同的地方，把这些地方渲染到真实的 DOM 中，进行了视图按需的更新。
  ```js
  var app = new Vue({
    el: '#app',
    data: {
      message: 'Hello Vue!'
    },
    render: function(createElement) {
      return createElement('div', {
        attrs: {
          id: 'app'
        }
      }, this.message)
    }
  })
  ```

## 输出

- 写了一篇博客：掘金[GIS 图层概念和总结](https://juejin.im/post/5d3ff32ef265da03986bcd21)，成就：阅读量：322，点赞：9，获得了 3 个关注者。继续💪
- [x] 按时 code review，借用了 [fork](https://git-fork.com/) 软件。
- [x] 选择下拉树的模糊搜索过程的思路记录。
  - 第一步，需要扁平化树的结构，然后直接对于搜索的值进行比对，匹配到一系列相关的树节点 filterNodes。
  - 第二步，找到 filterNodes 节点的父节点 parentNodes 和子节点
  - 第三步，合并所有的节点，filterNodes + pranetNodes + childNodes ，剔除重复的节点得到 processNodes
  - 第四步，对得到的树节点组合进行重新构建，生成树结构，进行渲染即可。
  代码
  ```js
  fuzzySearch(val) {
      const filterNodes = this.flatState.filter(obj =>
        obj.node.title.includes(val)
      );
      // 找到所有子节点的父节点，添加进去
      const parentNodes = [];
      if (filterNodes.length > 0) {
        filterNodes.forEach(node => {
          const nodes = this.getTreeUp(node.nodeKey);
          nodes.forEach(item => {
            if (!parentNodes.includes(item)) {
              parentNodes.push(item);
            }
          });
        });
      }

      // 找到所有节点的子节点
      let childNodes = [];
      filterNodes.forEach(node => {
        const children = node.children;
        if (children) {
          children.forEach(item => {
            const childNode = this.flatState.find(
              treeNode => treeNode.nodeKey === item
            );
            childNodes.push(childNode);
          });
        }
      });
      let data = [...filterNodes, ...parentNodes, ...childNodes];
      // 去重
      data = arrUnique(data, "nodeKey");
      // console.log(data);
      // 处理为渲染要求的树数据
      const processNodes = data.map(obj => {
        const newObj = {};
        newObj.parent = obj.parent !== undefined ? obj.parent : -1;
        return Object.assign(newObj, obj.node);
      });
      // console.log(processNodes);
      this.queryTree = this.buildTree(processNodes);
      this.stateTree = this.buildTree(processNodes);
      this.queryTreeflatState = this.compileFlatState(this.queryTree);
    },
  ```
- [x] Vue 双向绑定应用
  - 除了常见的 v-model，也可以考虑用 .sync 详细看官网，还有就是将父组件的数据包装成对象传递给子组件，因为JavaScript中的对象和数组是通过引用传入的，所以对于一个数组或对象类型的 prop 来说，在子组件中改变这个对象或数组本身将会影响到父组件的状态。这个也是选择框和下拉框勾选、删除时的响应式原理。

## 生活

- [x] 坚持仰卧起坐锻炼 90天。
- [x] 打了一场篮球，出了很多汗，生命在于运动，特别是程序员。

## 总结

多点产品思维，多把遇到的问题，抽时间去研究，分享输出文章，获得鼓励，形成良好的闭环。
