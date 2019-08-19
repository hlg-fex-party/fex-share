title: Summary of Interview
speaker: changzhn
prismTheme: solarizedlight

<slide :class="size-40 aligncenter">

# Summary of Interview {.text-landing.text-shadow}

By changzhn {.text-intro}

2019.8.17


<slide :class="size-40 aligncenter">

## 目录
---

<nav class="aligncenter">
* [ES6继承](./#slide=3)
</nav>

<slide :class="size-40 aligncenter">

### ES6的继承

---

```js {.animated.fadeInUp}
class Super {
    constructor(superName) {
        this.superName = superName;
    }

    static say() {
        return '666';
    }
}

class Sub extends Super {
    constructor(subName) {
        this.subName = subName;
    }
}

Sub.say(); // -> 666
```

<slide :class="size-40">

### 如何使用ES5实现ES6的继承

---

要实现的功能：
1. 子类实例可以调用父类原型上的方法；
2. 子类调用父类的静态方法；



<slide :class="size-40 aligncenter">

### 看下babel是如何实现

---

```js
"use strict";

function _inherits(subClass, superClass) {
  if (
      typeof superClass !== "function" 
      && superClass !== null
    ) {
    throw new TypeError("Super expression must either be null or a function");
  }

  subClass.prototype = 
    Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            writable: true,
            configurable: true
        }
    });
  if (superClass) _setPrototypeOf(subClass, superClass);
}

function _setPrototypeOf(o, p) {
  _setPrototypeOf = Object.setPrototypeOf || 
    function _setPrototypeOf(o, p) {
        o.__proto__ = p;
        return o;
    };
  return _setPrototypeOf(o, p);
}
```


<slide :class="size-40 aligncenter">

Bye