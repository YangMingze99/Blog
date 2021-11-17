---
title: 蛮好用的API - IntersectionObserver
date: 2021-11-18 01:04:12
tags: JavaScript
categories: JavaScript
excerpt: IntersectionObserver API 提供了一种异步观察目标元素与其祖先元素或顶级文档视窗 (viewport) 交叉状态的方法，性能较好。可以用来实现曝光埋点、懒加载、加载更多、吸底吸顶效果。
---
# 蛮好用的API - IntersectionObserver

## Part 1  三个大聪明

从前啊 有一个需求它叫商品曝光ctr埋点上报时机优化...

商品曝光埋点嘛 那不就是商品卡出现在可视区域就上报下信息嘛..  先搞一个判断是不是在可视区域的逻辑！

很快啊

大聪明[0]甩出了自己的代码

```javascript
function isInViewWindow(el){
    const viewPortHeight = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight;
    const offsetTop = el.offsetTop;
    const scrollTop = document.documentElement.scrollTop;
    cosnt top = offsetTop - scrollTop;
    return top <= viewPortHeight;
}
```

大聪明[0]：这个整挺好 都嘎嘎兼容  就是document写的有点累

大聪明[1]此时开始发炎：诶 你这多草率 快看看之前的代码有人用到了传说中的**Element.getBoundingClientRect()**

> `**Element.getBoundingClientRect()**` 方法返回元素的大小及其相对于视口的位置。

```javascript
function isInViewWindow(el){
    const viewWidth = window.innerWidth || document.documentElement.clientWidth;
    const viewHeight = window.innerHeight || document.documentElement.clientHeight;
    const {
        top,
        left,
        bottom,
        right
     } = el.getBoundingClientRect();
     return top>=0 && left >= 0 && right <= viewWidth && bottom <=viewHeight;
}
```

大聪明[2]：诶  我怎么不光看到**getBoundingClientRect**还看到了这个东西**IntersectionObserver**

这是啥？

## Part 2  IntersectionObserver 介绍及用法

**Intersection Observer** API 提供了一种异步检测目标元素与祖先元素或 [viewport](https://developer.mozilla.org/zh-CN/docs/Glossary/Viewport) 相交情况变化的方法。过去，相交检测通常要用到事件监听，并且需要频繁调用 [`Element.getBoundingClientRect()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect) 方法以获取相关元素的边界信息。事件监听和调用 [`Element.getBoundingClientRect()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect) 都是在主线程上运行，因此频繁触发、调用可能会造成性能问题。这种检测方法极其怪异且不优雅。

当然上面是MDN文档说的一部分，简单来说就是：保证性能的前提下**监听目标元素与其祖先或视窗的交叉状态**。

使用API当然要考虑兼容问题，好在这是2016年Chrome率先推出的API，目前的兼容性还不错当然**IE**除外。

官方同时提供了 [polyfill](https://github.com/w3c/IntersectionObserver/tree/master/polyfill)提高兼容性，这样我们就可以在项目中愉快的玩耍了

{% asset_img 兼容性.png %}

IntersectionObserver使用方式也很简单

```javascript
let io = new IntersectionObserver(callback, options);
```

`IntersectionObserver`支持两个参数：

1. `callback`是当被监听元素的可见性变化时，触发的回调函数

   接收一个参数entries，即IntersectionObserverEntry实例。描述了目标元素与root的交叉状态。具体参数如下：

   | 属性                  | 说明                                                         |
   | --------------------- | ------------------------------------------------------------ |
   | boundingClientRect    | 返回包含目标元素的边界信息，返回结果与element.getBoundingClientRect() 相同 |
   | **intersectionRatio** | 返回目标元素出现在可视区的比例                               |
   | intersectionRect      | 用来描述root和目标元素的相交区域                             |
   | **isIntersecting**    | 返回一个布尔值，下列两种操作均会触发callback：1. 如果目标元素出现在root可视区，返回true。2. 如果从root可视区消失，返回false |
   | rootBounds            | 用来描述交叉区域观察者(intersection observer)中的根.         |
   | target                | 目标元素：与根出现相交区域改变的元素 (Element)               |
   | time                  | 返回一个记录从 IntersectionObserver 的时间原点到交叉被触发的时间的时间戳 |

2. `options`是一个配置参数，可选项，有默认的属性值，包含三个属性

3. | 属性       | 说明                                                         |
   | ---------- | ------------------------------------------------------------ |
   | root       | 所监听对象的具体祖先元素。如果未传入值或值为null，则默认使用顶级文档的视窗(一般为html)。 |
   | rootMargin | 计算交叉时添加到**根(root)\**边界盒[bounding box](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FGlossary%2Fbounding_box)的矩形偏移量， 可以有效的缩小或扩大根的判定范围从而满足计算需要。所有的偏移量均可用\**像素**(`px`)或**百分比**(`%`)来表达, 默认值为"0px 0px 0px 0px"。 |
   | threshold  | 一个包含阈值的列表, 按升序排列, 列表中的每个阈值都是监听对象的交叉区域与边界区域的比率。当监听对象的任何阈值被越过时，都会触发callback。默认值为0。 |

​		rootMargin：10px 15px 20px 30px
{% asset_img viewport.jpg %}

*root元素只有在`rootMargin`为空的时候才是绝对的视窗*

## Part3 实际应用

### 1.曝光埋点

```javascript
const boxList = [...document.querySelectorAll('.box')]
var io = new IntersectionObserver((entries) =>{
  entries.forEach(item => {
    // intersectionRatio 曝光判定根据业务需求确定这里以50%为定
    if (item.intersectionRatio === 0.5) {
      // 埋点曝光代码或者props传递进来的onshow方法
      io.unobserve(item.target) //用完就丢掉 给浏览器减负一下
    }
  })
}, {
  threshold: [0.5], // 阀值设为0.5，当只有比例达到0.5时才触发回调函数
})
// observe遍历监听所有box节点
boxList.forEach(box => io.observe(box))

```

### 2.加载更多

```javascript
function loadMore() {
  const observer = new IntersectionObserver(
    (entries) => {
      const loadingEntry = entries[0]
      if (loadingEntry.isIntersecting) {
        // 发请求  拿到数据然后大家懂得都懂
      }
    },
    {
      rootMargin: '0px 0px 200px 0px', // 扩大viewport 提前加载 优化下用户体验
    },
  )

  observer.observe(document.querySelector('.mod_loading')) // 观察元素
}
```

### 3.元素吸顶、吸底

除了CSS粘性定位以外也可以用这个来实现，将需要吸顶/底的元素外包裹一层父元素并赋予高度占位，防止需要吸顶/底的元素固定时页面有抖动效果出现，并监听父元素状态

```javascript
function fixedELeMent(parentEle, fixEle) {
  const ele = fixEle
  const observer = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        ele.style.cssText = ''
      } else {
        ele.style.cssText = 'position: fixed; top: 0; left: 0'
      }
    })
  }, {
    threshold: 1, 
  })
  observer.observe(parentEle);
}
```

### 4.懒加载

```javascript
const imgList = [...document.querySelectorAll('img')]
var io = new IntersectionObserver((entries) =>{
  entries.forEach(item => {
    if (item.isIntersecting) {
      item.target.src = item.target.dataset.src
      io.unobserve(item.target)
    }
  })
}, {
	rootMargin: `0px 0px 50px 0px`,
})
imgList.forEach(img => io.observe(img))
```
## Part 4 参考

[Intersection Observer API](https://developer.mozilla.org/zh-CN/docs/Web/API/Intersection_Observer_API)

[Element.getBoundingClientRect()](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect)

[Position:sticky](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position)

