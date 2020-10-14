---
title: 关于eventListener的那些事
date: 2018-08-22 11:41:52
tags: [js, eventlLstener, chrome dev tool, web api, 事件冒泡]
category: WebAPI
---

> If multiple identical `EventListeners` are registered on the same `EventTarget` with the same parameters, the duplicate instances are discarded. They do not cause the `EventListener` to be called twice, and they do not need to be removed manually with the `removeEventListener()` method. [^1]

如果在同一个`EventTarget`上添加同一个（reference 相同）`EventListeners`，该 listener 只会执行一次，无需手动调用`removeEventListener`移除多余的 listener

所以在`addEventListener`时应尽量**避免使用匿名函数**的形式，而应该引用已定义好的函数，从而避免之后重复添加 listener 时被重复执行多次，并且也为`removeListener`提供可能性

## getEventListeners

有时候我们会想知道 document 里的全部或某个 object 上面都绑定了什么 listener，除了自己写一段代码去查看之外，chrome dev tools 也为我们提供了一些方便的接口[^3]

### 查看 document 中的所有 listener

https://gist.github.com/dmnsgn/36b26dfcd7695d02de77f5342b0979c7

### 使用 chrome 查看单个 object 上的 listener

直接在 chrome 的 console 中使用`getEventListeners`接口即可

```javascript
getEventListeners(document);
```

如果想知道 listener 的函数源码，可以对该 listener 右键选择 **store as global variable**， chrome 会将其储存为名为`tempX`的变量(X 为正整数)，并打印出该变量内容，即可看到 listener 函数的具体代码

## addEventListener

大多数时候我们在`addEventListener`时，都会省略第三个参数`options`。要了解`options`中的各项参数有什么用途，首先我们需要了解，事件冒泡(Event Bubbling)和事件捕捉(Event Capturing)的执行顺序[^4]。

### 事件冒泡 事件捕捉

![img](https://mdn.mozillademos.org/files/14075/bubbling-capturing.png)

当触发一个 event 时，现代浏览器可以有两种执行模式/顺序，默认为**事件冒泡**

- 事件冒泡，由下至上，child 向 ancestor 依次查找 eventListner 并执行
- 事件捕捉，由顶至下，由最外层 ancestor 向 child 依次查找、执行， (until it reaches the element that was actually clicked on)

### `options`[^5]

`options`包括以下几个选项：

- `capture`，如果为`true`，该 event 会最先 dispatch。 ** before any other children doms' same event ** 如同上一节所提到的，如果设置为`true`，浏览器会按照事件捕捉的顺序来执行相关 listener
- `once`，如果为`true`，listener 最多只会执行一次，之后会自动移除
- `passive`，如果为`true`，不会调用`preventDefault()`，如果该方法被调用，则该方法会被忽略且抛出警告 ⚠️。

```
Unable to preventDefault inside passive event listener invocation.
```

`passive`为`false`时，`touch`event 会阻塞浏览器的主线程，从而影响 scroll 的性能，因此一些浏览器(chrome, firefox)将`touchstart` `touchemove`事件在`window` `document` `document.body`级别的`passive`默认值设为`true` [^6]

MDN 的这个例子非常好的解释了各种 option 搭配下的各种情况
https://jsfiddle.net/HiiTea/m2vha9sj/embedded/result,html,js

另外需要注意的是，我们经常看到这样的写法：

```javascript
element.addEventListener('click', myClickHandler, false);
```

第三个参数并非一个`object`，而是一个`boolean`，这样的写法是为了兼容一些旧的浏览器，当使用这样的写法时，第三个参数`false`指代的是`capture`参数

> The options argument sets listener-specific options. For compatibility this can be a boolean, in which case the method behaves exactly as if the value was specified as options’s capture. [^7]

### IE

IE9 之前，没有`addEventListener`方法，需要使用`attachEvent`方法代替，该方法不接受第三个参数 [^8]

## removeListener

> While `addEventListener()` will let you add the same listener more than once for the same type if the options are different, the only option `removeEventListener()` checks is the capture/useCapture flag. Its value must match for `removeEventListener()` to match, but the other values don't. [^9]

这个就是说，如果在`addEventListener`是添加了某些`options`，那么移除时也需要添加相应的`options`才能成功移除 listener 😭 真是麻烦

```javascript
document.addEventListener('click', handler, { passive: true });
document.removeEventListener('click', handler); // fails
```

[^1]: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Multiple_identical_event_listeners#Multiple_identical_event_listeners
[^3]: https://developers.google.com/web/tools/chrome-devtools/console/command-line-reference
[^4]: https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#Event_bubbling_and_capture
[^5]: https://developers.google.com/web/updates/2016/10/addeventlistener-once
[^6]: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Improving_scrolling_performance_with_passive_listeners
[^7]: https://dom.spec.whatwg.org/#interface-eventtarget
[^8]: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Legacy_Internet_Explorer_and_attachEvent
[^9]: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener
