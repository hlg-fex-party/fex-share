title: 2019/08/18 share
speaker: 张文斐
plugins:
    - echarts

<slide class="bg-black-blue aligncenter" image="https://source.unsplash.com/C1HhAQrbykQ/ .dark">

# 2019/08/18{.text-landing.text-shadow}

简单实现一个 computed {.text-intro.animated.fadeInUp.delay-200}

By 张文斐 {.text-intro.animated.fadeInUp.delay-500}

<slide :class="size-60 aligncenter">

### Computed 依赖收集原理{.aligncenter}

---

如何知道 Computed 都依赖了哪些已观察的变量？

一句话：Computed 函数执行时，会触发已观察变量 getter 函数{.tobuild.animated.heartBeat}

<slide :class="size-60 aligncenter">

既然已经知道 computed 是如何收集依赖的，那么要实现一个简单的 computed 就很简单了

<slide :class="size-100 aligncenter">

### Computed 两步走
##### Part1：收集依赖
执行 computed 时，将此 computed 进行缓存
==如果 computed 中有依赖，会触发对应依赖的 getter==

:::column {.vertical-align}
```js
// 此处演示为了精简代码只演示一个变量
const Dep = { subs: { number: [] }, target: null }
const v = {
  data: { number: 1 },
  computed: {
    count() {
        return this.data.number + 1
    }
  }
}

Dep.target = v.computed.count;
Dep.target();
```
---
```js
Object.defineProperty(v.data, 'number', {
  get: function() {
    // 左边 count 执行时，会先读取 this.data.number 的值
    // 所以会触发这里的getter
    if (Dep.target) {
      // 通过Dep.target可以知道触发的 computed 是哪个
      Dep.subs.number.push(Dep.target)
    }
    return value;
  },
})
```
:::

<slide :class="size-60 aligncenter">

##### Part2：依赖变化重新计算 computed

---

依赖变化时会触发 setter，在这个时候执行所有对应的 computed，即可刷新 computed 的值

```js
// 当依赖发生变化时
v.data.number = 10;

// 触发 setter
Object.defineProperty(v.data, 'number', {
  set: function() {
    ...
    Dep.notice();
  },
})

// 重新计算 computed
Dep.notice = function() {
  // 遍历所有依赖这个变量的 computed，所有的都重新计算一遍
  Dep.subs.number.forEach(item => item());
}
```

<slide :class="size-60 aligncenter">

### Computed 完整简单版

##### 初始化参数

```js {.animated.fadeInUp}
    window.v = {
        data: {
            apple: 3,
            water: 3,
            smoke: 20,
        },

        computed: {
            count() {
                return this.apple + this.water + this.smoke
            },
            price() {
                return this.apple * 20 + this.water * 5 + this.smoke * 30
            }
        }
    };
```

<slide :class="size-60 aligncenter">

##### 初始化Dep

 用来收集依赖以及在依赖发生变化时重新计算对应 computed 的值

```js {.animated.fadeInUp}
    const Dep = initDep();

    function initDep() {
        const Dep = {
            target: null,
            subs: {}
        };

        /*
        * 依赖收集
        * @param {string} key data的key
        * */
        Dep.depend = function(key) {
            if (!this.subs[key]) {
                this.subs[key] = [];
            }

            const subs = this.subs[key];
            if (subs.indexOf(item => item.getter === this.target.getter) === -1) {
                subs.push(this.target);
            }
        };

        /*
        * 重新计算
        * @param {string} key 改变的依赖项
        * */
        Dep.notify = function(key) {
            this.subs[key].forEach(item => item.getter());
        };

        return Dep;
    }
```

<slide :class="size-60 aligncenter">

##### 初始化data
---

```js {.animated.fadeInUp}
    initData();

    function initData() {
        for(let [key] of Object.entries(v.data)) {
            // 将 data 代理到 v
            proxyData(key, v.data[key]);
            // 监听 data
            defineData(key, v.data[key]);
        }
    }
```

<slide :class="size-60 aligncenter">

##### 代理data

目的是在 computed 中通过 this.xxx 访问到 v.data.xxx，此举锦上添花，并不重要

```js {.animated.fadeInUp}
    /*
    * 将data下的变量挂载到根节点
    * @param {string} key 代理的变量名
    * @param {*} value 对应的值
    * */
    function proxyData(key, value) {
        const obj = v.data;
        v[key] = value;
        const options = {
            enumerable: true,
            configurable: true,
            get: function() {
                return obj[key];
            },
            set: function(newValue) {
                obj[key] = newValue;
            }
        };

        Object.defineProperty(v, key, options)
    }
```

<slide :class="size-60 aligncenter">

##### 观察data

---

```js {.animated.fadeInUp}
    /*
    * 劫持v.data中变量的get和set
    * */
    function defineData(key, value) {
        Object.defineProperty(v.data, key, {
            configurable: true,
            enumerable: true,
            get: function() {
                if (Dep.target) {
                    // 收集依赖
                    Dep.depend(key);
                }
                return value;
            },
            set: function(newValue) {
                if (newValue === value) return;
                value = newValue;
                Dep.notify(key);
            }
        })
    }
```

<slide :class="size-60 aligncenter">

##### 初始化computed

---

```js {.animated.fadeInUp}
initComputed();

function initComputed() {
  for(let [key, value] of Object.entries(v.computed)) {
    if (key in v.data) {
      // 保证data中没有变量名和computed中的变量名重复
      throw new Error(`The computed property "${key}" is already defined in data.`)
    }

    Dep.target = {
      key,
      getter: function() {
        v[key] = value.call(v);
      }
    };

    Dep.target.getter();
  }
}
```

<slide :class="size-90 aligncenter">
##### 小结

---

初始化 computed 的时候，把 computed 缓存到一个变量中 {.animated.fadeInUp.delay-200}

从而在触发依赖的 getter 时，通过检查缓存变量，我们就能够知道谁依赖了哪个 computed {.animated.fadeInLeft.delay-500}

至于剩下的，将 computed 初始化的值代理到根节点，依赖的值变更后重新计算 computed 的值，都很简单了 {.animated.fadeInBottom.delay-800}

<slide :class="size-60 aligncenter">
Bye {.animated.fadeInUp}