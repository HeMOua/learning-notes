# Vuex

## vuex简介

+ 官方解释：Vuex是一个专为Vuejs应用程序开发的状态管理模式。
  + 它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。
  + Vuex 也集成到Vue的官方调试工具devtools extension，提供了诸如零配置的time-travel 调试、状态快照导入导出等高级调试功能。
+ 状态管理到底是什么？
  + 状态管理模式、集中式存储管理这些名词听起来就非常高大上，让人捉摸不透。
  + 其实，你可以简单的将其看成把需要多个组件共享的变量全部存储在一个对象里面。
  + 然后，将这个对象放在顶层的Vue实例中，让其他组件可以使用。
  + 那么，多个组件是不是就可以共享这个对象中的所有变量属性了呢？
+ 为什么官方还要专门出一个插件vuex呢？难道我们不能自己封装一个对象来管理吗？
  + 当然可以，只是我们要先想想VueJS带给我们最大的便利是什么呢？没错，就是响应式。
  + 如果你自己封装实现一个对象能不能保证它里面所有的属性做到响应式呢？当然也可以，只是自己封装可能稍微麻烦一些。
  + 不用怀疑，Vuex就是为了提供这样一个在多个组件间共享状态的插件，用它就可以了。
+ 但是，有什么状态时需要我们在多个组件间共享的呢？
  + 如果你做过大型开放，你一定遇到过多个状态，在多个界面间的共享问题。
  + 比如用户的登录状态、用户名称、头像、地理位置信息等等。
  + 比如商品的收藏、购物车中的物品等等。
  + 这些状态信息，我们都可以放在统一的地方，对它进行保存和管理，而且它们还是响应式的

**状态管理**

- **state**，驱动应用的数据源；
- **view**，以声明方式将 **state** 映射到视图；
- **actions**，响应在 **view** 上的用户输入导致的状态变化。

![img](img/flow.png)

![vuex](img/vuex.png)

+ 全局单例模式（大管家）
  + 我们现在要做的就是将共享的状态抽取出来，交给我们的大管家，统一进行管理。
  + 之后，每个视图，按照我规定好的规定，进行访问和修改等操作。
  + 这就是Vuex背后的基本思想。

## 简单案例

目标：两个按钮控制变量count的大小并显示在页面上

1. 安装依赖

   ```shell
   npm i vuex --save
   ```

2. 新建一个文件 `store/index.js` 内容如下

   ```js
   import Vue from 'vue'
   import Vuex from 'vuex'
   
   // 1.安装插件
   Vue.use(Vuex)
   
   // 2.创建对象
   const store = new Vuex.store({
       state: {}, // 存放所有的数据
       mutations: {}, // 操作 state 中的数据
       actions: {}, // 进行异步操作，间接调用 mutations 中的方法
       getters: {}, // 
       modules: {} // 
   })
   
   // 3.导出store对象
   export default store
   ```

3. vuex 挂载到 Vue 上

   main.js

   ```js
   import store './store'
   
   new Vue({
   	...
       store
   })
   ```

4. 页面

   ```vue
   </template>
   <div>   
       <div>{{ $store.state.count }}</div>
       <button @click="increase">增加</button> | 
       <button @click="decrease">减少</button>
   </div>
   </template>
   
   <script>
   export default {
     name: 'App',
     methods: {
       increase(){
         this.$store.commit('increase')
       },
       decrease(){
         this.$store.commit('decrease')
       }
     }
   }
   </script>
   ```

5. store

   ```js
   export default new Vuex.Store({
     state: {
       count: 1000
     },
     mutations: {
       increase(state){
         state.count++
       },
       decrease(state){
         state.count--
       }
     },
     actions: {
     },
     modules: {
     }
   })
   ```

## Vuex 核心概念

### 概念速记

1. 对于`state`，要做的就是赋予他一个数据对象，类似vue页面的data
2. `getters`：其中有一系列的getter方法，每个getter方法可以**接受3个参数**。第一个参数就是当前模块的`state`；第二个参数就是`getters`，这个getters可以调用当前模块的其他getter方法；第三个参数就是`rootState`，代表父模块的state（较少用），他只有在使用了modules时才能使用
3. `mutations`：其中有一系列的方法，每个方法可以**接受2个参数**。第一个参数就是当前模块的`state`，和getters方法的第一个参数一样；第二个参数就是commit方法传递过来的数据，如果commit传递的是多个数据的话，那么这个数据必须为一个对象
4. `actions`：其中也有一系列方法，每个方法**接受2个参数**。第一个参数为context，也就是上下文对象，类似vue页面的`this.$store`，通过context可以访问commit、dispatch、state，所以一般使用es6花括号解构使用。如果使用了modules，那么这个context还存在 `rootState`和 `rootGetters` 这两个，分别是是父模块的 state 和 getters；第二个参数就是所传递的数据，类似mutations，传递多个数据时这个数据必须为一个对象

### State

**单一状态树**，英文名 `Single Source of Truth`，也可以翻译成单一数据源。

用一个生活中的例子做一个简单的类比。

+ 我们知道，在国内我们有很多的信息需要被记录，比如上学时的个人档案，工作后的社保记录，公积金记录，结婚后的婚姻信息，以及其他相关的户口、医疗、文凭、房产记录等等（还有很多信息）。
+ 这些**信息被分散在很多地方进行管理**，有一天你需要办某个业务时（比如入户某个城市），你会发现你需要到各个对应的工作地点去打印、盖章各种资料信息，最后到一个地方提交证明你的信息无误。
+ 这种保存信息的方案，不仅仅低效，而且不方便管理，以及日后的维护也是一个庞大的工作（需要大量的各个部门的人力来维护，当然国家目前已经在完善我们的这个系统了）。

这个和我们在应用开发中比较类似：

+ 如果你的状态信息是保存到多个Store对象中的，哪么之后的管理和维护等等都会变得特别困难。
+ 所以Vuex也使用了单一状态树来管理应用层级的全部状态。
+ 单一状态树能够让我们最直接的方式找到某个状态的片段，而且在之后的维护和调试过程中，也可以非常方便的管理和维护。

我们会将项目==所有用到的共享变量全部放在 state 中==

### Getters

#### 案例

类似计算属性，也就我们每次获取的值都是在对 state 中的数据做一些相应的处理再返回，如下获取 count 的平方

**store/index.js**

```js
export default new Vuex.Store({
  state: {
    count: 1000
  },
  getters: {
    pow(state){
        return state.count * state.count
    }
  },
  mutations: {},
  actions: {
  },
  modules: {
  }
})
```

**App.vue**

```vue
<div>
    {{$store.getters.pow}}
</div>
```

#### 参数

1、调用已有的 getters

```js
export default new Vuex.Store({
    state: {
        students: [
            { name: '小明', age: 18 },
            { name: '黄明', age: 21 },
            { name: '大明', age: 32 },
        ]
    },
    getters: {
        oldStudent(state){
            return state.students.filter(s => s.age > 20)
        },
        oldStudentLength(state, getters){
            return getters.oldStudent.length
        }
    },
    mutations: {},
    actions: {
    },
    modules: {
    }
})
```

2、getters 方法本身带参数

**App.vue**

```vue
<div>{{ $store.getters.moreAgeStudent(10) }}</div>
```

**store/index.js**

```js
export default new Vuex.Store({
    state: {
        age: 28,
        students: [
            { name: '小明', age: 25 },
            { name: '黄明', age: 26 },
            { name: '大明', age: 27 },
        ]
    },
    getters: {
        moreAgeStudent(state) {
            return function(age){
                return state.students.filter(s => {
                    return s.age > age
                })
            }
            // 简写
            return age => state.students.filter(s => s.age > age)
        },
    }
})
```

### Mutations

Mutation主要包括两部分：

+ 字符串的事件类型（type）
+ 一个回调函数（Handler），该回调函数的第一个参数就是state。

#### 传递参数

**App.vue**

```vue
<script>
export default {
  name: 'App',
  data(){
    return {
      span: 5,
      name: '',
      age: ''
    }
  },
  methods: {
    increase(count){
      this.$store.commit('increase', Number(count))
    },
    decrease(count){
      this.$store.commit('decrease', Number(count))
    },
    add(){
      this.$store.commit('addUser', {name: this.name, age: this.age})
    }
  }
}
</script>
```

**store/index.js**

```js
export default new Vuex.Store({
  state: {
    age: 28,
    students: [
      { name: '小明', age: 25 },
      { name: '黄明', age: 26 },
      { name: '大明', age: 27 },
    ]
  },
  mutations: {
    increase(state, count) {
      state.age += count
    },
    decrease(state, count) {
      state.age -= count
    },
    addUser(state, user){
      state.students.push(user)
    }
  }
})
```

`commit('handle', payload)`的第二个参数就是有效负载，用来传递参数的

==若想传递多个参数，就需要传递一个对象==

#### 提交风格

1、之前使用的 `this.$store.commit('decrease', Number(count))`是一种普通的提交风格

传递到 vuex 的 increase 方法时，count 就代表所传递的 count

```js
mutations: {
    increase(state, count) {
        state.age += count
    }
}
```

2、而另一种提交风格就是

```js
this.$store.commit({
    type: 'decrease'
    count: Number(count)   
})
```

那么vuex对应方法收到的参数 count 就是 commit 的整个对象了，

如下的 count 应该等于  `{type: 'decrease', count: xxx}`

```js
new Vue.store({
    ...
    mutations: {
        increase(state, count) {
            state.age += count.count
        }
    },
})    

// 合理的应该如下写法
new Vue.store({
    mutations: {
        increase(state, payload) {
            state.age += payload.count
        }
    }
})
```

#### 响应规则

所有定义在 state 中的数据都会被加入到响应式系统中，而响应时系统会监听属性的变化，当属性发生变化时，会通知所有界面中用到该属性的地方，让界面发生刷新

但是如果是通过 js **动态添加上到 state 的变量**就==不会==被加入到响应式系统中，如下操作就是无效果的

```js
new Vue.store({
	...
    mutations: {
        updateInfo(state){
            state.info['address'] = '北京'
        },
        deleteInfo(state){
            delete state.info.age
        }
    }
})
```

若想要动态将元素添加到响应式系统中，必须通过以下方法

**store/index.js**

```js
new Vue.store({
	...
    mutations: {
        updateInfo(state){
            Vue.set(state.info, 'address', '北京')
        },
        deleteInfo(state){
            Vue.delete(state.info, 'age')
        }
    }
})
```

+ 通常情况下，Vuex要求我们Mutation中的方法必须是同步方法.
  + 主要的原因是当我们使用devtools时，可以devtools可以帮助我们捕捉mutation的快照。
  + 但是如果是异步操作，那么devtools将不能很好的追踪这个操作什么时候会被完成。

### Actions

+ 我们强调，不要再Mutation中进行异步操作。
  + 但是某些情况，我们确实希望在Vuex中进行一些异步操作，比如网络请求，必然是异步的.这个时候怎么处理呢？
  + Action类似于Mutation，但是是用来代替Mutation进行异步操作的.

#### 简单案例

**store/index.js**

```js
new Vue.store({
    actions: {
        aUpdateInfo(context, payload){
            setTimeout(() => {
                context.commit('updateInfo')
				console.log(layload.msg)
                layload.success()
            }, 1000)
        }
    }
})
```

**App.vue**

```vue
<script>
    export default{
        methods: {
            updateInfo(){
                this.$store.dispatch('aUpdateInfo', {
                    msg: {},
                    success: () => {
                        console.log('完成')
                    }
                })
            }
        }
    }
</script>
```

#### 结合Promise

**store/index.js**

```js
new Vue.store({
    actions: {
        aUpdateInfo({ commit }, payload){
            return new Promise((reslove, reject) => {
                setTimeout(() => {
                    commit('updateInfo')
                    console.log(layload)
                    resolve('111')
                }, 1000)
            })
        }
    }
})
```

**App.vue**

```vue
<script>
    export default{
        methods: {
            updateInfo(){
                this.$store.dispatch('aUpdateInfo', {
                    name: {},
                    age: 20
                }).then(res => {
                    ...
                })
            }
        }
    }
</script>
```

### Modules

```js
const module1 = {
    state: { name: '张三' },
    getters: {
        fullname(state, getters, rootState){
            return state.name + '111'
        }
    },
    mutations: {
        updateName(state){

        }
    },
    actions: {
        aUpdateInfo(context){
            
        }
    }
}

export default new Vuex.Store({
    state: {
        age: 28,
        students: [
            { name: '小明', age: 25 },
            { name: '黄明', age: 26 },
            { name: '大明', age: 27 },
        ]
    },
    mutations: {
    },
	modules: {
        a: module1
    }
})
```

需要注意的有一下几点

+ 模块中的 getters 方法接受三个参数 
  + `state`：表示当前模块的 state
  + `getters` ：当前模块的 getters
  +  `rootState`：父模块的 state
+ 模块中的 actions 方法
  + 本身接受一个 context，但是这个context 不再像父模块代表 `new Vue.store`了，而是代表着当前模块的上下文对象
  + 模块的 context 对象包含两个特殊的对象 `rootState`和 `rootGetters` 是父模块的 state 和 getters

其他便没有什么变化，在组件中可以像调用根模块的 getters 、mutations 和 actions 一样来使用

