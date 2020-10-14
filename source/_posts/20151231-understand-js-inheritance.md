---
title: 对JS类和继承的一些理解
date: 2015-12-31 11:36:54 +8
tags: [JS, prototype, 继承, 原型]
category: JS
---

本文只总结最常用的实现方法，不具体讨论各种实现方法的优缺点（这些具体可以看[《JavaScript 高级程序设计》](http://book.douban.com/subject/10546125/)）。

直到 ES5，JS 也还是一个没有类的语言，虽然 ES6 中可以使用 class 关键字，但据说也只是语法糖。(不知道好不好吃 ԅ(¯﹃¯ԅ))

### 类的实现

类的两个基本元素就是**属性**和**方法**。

JS 中类的实现有很多种，构造函数模式、原型模式等等，各有各的优缺点，最常用的是混合了构造函数和原型模式的混合模式。

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHi = function () {
  console.log('Hi, ' + this.name);
};
```

使用构造函数来构造**属性**，然后往原型对象添加**方法**。这样的好处是：

- 每个实例之间不用共享属性，拥有各自独立的属性。也就可以避免当属性为引用类型（数组、对象）时，修改一个实例的属性会影响到其他实例的情况。
- 每个实例之间共用原型对象上的方法，实现了函数复用。

### 继承的实现

JS 中继承的实现也有很多种，借用构造函数，原型链等等。
子类继承父类，当然继承的就是父类的**属性和方法**。JS 中继承的实现，最常用的也是混合模式。

```js
function Student(name, age) {
  Person.call(this, name);
  this.age = age;
}

console.log(Student.prototype.constructor); // Student

Student.prototype = new Person();
// 补充！有时候我们也会见到如下方式，具体区别请看下一小节
Student.prototype = Object.create(Person.prototype);

console.log(Student.prototype.constructor); // Person

Student.prototype.constructor = Student; // 重写constructor

Student.prototype.sayAge = function () {
  console.log("I'm " + this.age);
};
```

使用**借用构造函数**的方式来继承**属性**，然后使用**原型链**来继承**方法**。通过将子类的原型对象指向父类的实例，子类的实例就可以通过原型链向上查找到父类原型上的方法。

#### `Object.create` vs `new`

有时候我们也会见到如下的继承方式：

```js
Student.prototype = Object.create(Person.prototype);
```

Object.create - 不调用构造函数, 原型 prototype 上不会存在`undefined`的属性，（构造函数中的属性不会存在于原型链上，除非原型链上定义了该属性/方法，例如`Person.prototype.sayhi = function(){}`）
new - 调用构造函数, 但由于`Student.prototype = new Person();`没有传入参数，因此`Student.prototype.name === undefined`

举个 🌰
https://jsfiddle.net/HiiTea/cfy89tru/embedded/js
可以看到，student 的原型链上多了一个`age: undefined`属性
![image](https://user-images.githubusercontent.com/36024221/46127882-8f360380-c264-11e8-82af-69f214fe66cc.png)

### 重写子类构造函数的意义

`Student.prototype = new Person();`这一步完全改变了 Student 原型对象的引用，`Student.prototype.constructor` 变为了 Person 原型对象的 constructor。
~~个人觉得重写`Student.prototype.constructor = Student;`没有什么实际意义，可能只是**约定俗成的一种潜规则**。
人们通常可能已经习惯了使用 new 操作符的时候，构造函数的一致性~~

== update 2019/07/11 ==
啊，4 年的前的我果然 too simple too naive
重写`Student.prototype.constructor = Student;`除了能保持构造函数一致性的问题，还能使`instanceof`运算符给出正确的运算结果。

如果没有重写，`student instanceof Person` 的结果将会是`false`，因为`instanceof`的内部机制是，**判断对象的原型链中是否能找到对应的`prototype`**
== update end ==

```js
// Student.prototype.constructor = Student; 在上面的代码中注释掉这一句
var xiaobai = new Student('小白妹妹', 10);
xiaobai.constructor; // Person
```

当你在代码中遇到上面这种情况，如果不去查看之前的代码的话，肯定会觉得奇怪，为什么明明通过 Student 构造函数 new 了一个 Student 实例，而这个实例，却说自己的构造函数是 Person？！？！WTF？！

### 参考

[1] http://stackoverflow.com/questions/4012998/what-it-the-significance-of-the-javascript-constructor-property
