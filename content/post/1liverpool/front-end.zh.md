---
title: "前端刷题笔记"
date: 2023-05-07T08:42:27Z
draft: false
author: ["Martin"]
tags: 
- 前端
description: "记录刷前端时遇到的知识点"
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: "front-end"
comments: true
showToc: true # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: ""
    caption: ""
    alt: ""
    relative: false
mermaid: true
---
# 2023年5月7日
1. 为同一个文件添加多种文件格式
```html
<video controls>
    <source src="html_5.mp4" type="video/mp4">
    <source src="html_5.ogv" type="video/ogg">
    Your browser does not support the video tag.
</video>
```
为什么要添加多种格式？
- 解决备份支持和媒体支持

什么是备份支持和媒体支持？
- 备份支持：提供替代方案，一般是相同的浏览器，版本更老的一代需要这个替代方案。
- 媒体支持：需要提供多种格式的支持来适应不同的浏览器。

2. 标签
    - ```<audio>``` 与 ```</audio>``` 之间插入的内容不是用来解释控件的，而是在浏览器不支持audio标签时显示的文字。
    - 当 ```<video>``` 标签中包含 controls 属性时，浏览器将自动为视频播放器提供一组默认的控制选项。用户可以根据自己的需求控制视频的播放。
    - ```<datalist>``` 标签与 ```<input>``` 标签结合使用，可以为用户提供一个预定义的选项列表。
    ```html
    <input list="fruits" id="fruit" name="fruit">
        <datalist id="fruits">      
            <option value="Apple">
            <option value="Banana">
        </datalist>
    ```
    - 当 ```<progress>``` 标签没有设置 max 和 value 属性时，它会显示一个不确定进度的滚动条。在这种情况下，进度条的动画将自动滑动，但无法显示具体的进度值。
    ```html
    <progress value="50" max="100"></progress>
    ```
3. DHTML
    - DHTML（Dynamic HTML）是一种将 HTML、CSS、JavaScript 等技术结合起来，以创建动态、交互式网页的技术组合。
    - DHTML实现了网页从Web服务器下载后无需再经过服务的处理，而在浏览器中直接动态地更新网页的内容、排版样式和动画的功能
4. DOM元素带有ID属性
    - 唯一性：ID 必须在整个 HTML 文档中是唯一的。如果有多个元素具有相同的 ID，将导致 JavaScript 在访问元素时出现问题，可能导致意外行为。因此，在为元素分配 ID 时，确保不会重复。
    - 样式污染：如果您的 CSS 使用 ID 选择器为元素应用样式，这可能导致样式污染问题。ID 选择器具有较高的优先级，这可能会导致其他 CSS 规则无法覆盖 ID 选择器的样式。要解决这个问题，您可以考虑使用 CSS 类选择器来应用样式，以便更容易地控制和覆盖样式。
    - JavaScript 性能：当使用 JavaScript 查询具有特定 ID 的元素时，最好使用 ```document.getElementById()``` 方法，因为这是最快的查询方式。使用其他查询方法，如 ```document.querySelector()``` 或 ```document.querySelectorAll()```，可能会导致较慢的查询性能，尤其是在大型的 DOM 结构中。
    - 与现有代码冲突：在开发大型应用程序或与其他人协作时，如果不注意命名约定和唯一性，给元素分配 ID 可能会导致与现有代码冲突。在这种情况下，可以采用一致的命名规则和确保 ID 唯一性的方法来避免潜在冲突。

# 2023年5月8日
今天刷了几道react的题，记录一下
1. state和props
    - state和props都可以控制组件的渲染输出
    - props和state都是普通的JavaScript对象
2. Error boundaries 错误边界
    - Error boundaries是一种react组件，这个组件可以捕获并处理子组件树发生的JS错误，当发生错误时，错误边界组件可以捕获错误，展示一个备用 UI，从而避免整个应用崩溃。
    - 错误边界仅适用于捕获组件渲染阶段、生命周期方法和构造函数中的错误。它无法捕获事件处理程序或异步代码中的错误。
    - 要将一个组件变成错误边界，需要在该组件内定义 ```componentDidCatch(error, info)``` 或 ```static getDerivedStateFromError(error)``` 生命周期方法之一。
    - componentDidCatch(error, info) 打印错误信息，如日志记录或向服务器报告错误。这个方法不会改变组件状态。
    - static getDerivedStateFromError(error) 方法用于在发生错误时更新组件状态，以便根据错误信息渲染备用UI。
3. React剪贴板事件
    - onCopy，onCut，onPaste
4. Portal的冒泡逻辑
    - Portal 是一种特殊的技术，它允许将子组件渲染到父组件以外的 DOM 节点
    - 当使用 ReactDOM.createPortal(child, container) 渲染子节点时，尽管子节点被渲染到父组件以外的 DOM 节点上，但它在 React 组件树中仍然被认为是父组件的子节点。
    - 当通过 Portal 进行事件冒泡时，事件将沿着 React 组件树向上冒泡，而不是 DOM 树。
    - 在事件冒泡过程中，事件将继续向父组件和其他祖先组件传播，就像子节点直接挂载在父组件内部一样。这是 React 事件系统的一个特性，它保证了组件树中的事件行为的一致性，而不受 DOM 结构的影响。
5. useEffect
    - useEffect()接收两个参数，一个是回调函数，一个是依赖项数组。只有当依赖项发生变化时，回调函数才执行
    - useEffect 可以模拟不同的生命周期方法(componentDidMount，componentDidUpdate, componentWillUnmount)

# 2023年5月13日
前几天在赶due，没时间刷题，终于空下来，又刷了几道题
1. JSX
    - 使用JSX写的代码最终会转换成使用React.createElement()的形式，React.createElement：创建并返回指定类型的新React元素。
2. React支持的触摸事件
    - React支持的触摸事件有onTouchCancel，onTouchEnd，onTouchMove，onTouchStart
3. Redux中的store
    - 不存在store.createStore方法，store可以使用Redux提供的createStore生成
    ```javascript
    store.getState() // 获取状态      
    store.dispatch({ type: xxx, data: xxx }) // 分发动作对象      
    store.subscribe(() => console.log('@')) // 设置监听函数，一旦 State 发生变化，就自动执行这个函数     
    ```