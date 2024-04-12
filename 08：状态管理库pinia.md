## 08：状态管理库Pinia

> `pinia`是一个vue的状态管理库。



### 8.1：安装与使用`pinia`

> 1. 安装语法：`npm install pinia`
> 2. 创建一个`pinia`（根存储）并将其传递给应用程序



**代码演示：**

```js
// main.js
......
import { createPinia } from 'pinia'

......

app.use(createPinia())
......
```



### 8.2：`store`

> 1. `store`是一个保存状态和业务逻辑的实体，他并不与你的组件树绑定；换句话说，它承载着全局状态；它有点像一个永远存在的组件，每个组件都可以读取和写入它。
>
> 2. `store`有三个概念，`state`、`getters`和`actions`，我们可以理解成组件中的`data`、`computed`、`methods`。
>
> 3. 在项目中的`src\store`文件夹下不同的`store.js`文件
>
> 4. `store`是用`defineStore( name, function | options )`定义的，建议其函数返回的值命名为`use...Store`方便理解
>
>    `a.` 参数`name`：名字，必填值且唯一
>
>    `b.` 参数`function | options`：可以是对象或函数形式
>
>    - 对象形式【选项模式】：其中配置`state`、`getters`、`actions`
>    - 函数形式【组合模式，类似组件组合式API的书写方式】，定义响应式变量和方法，并且`return`对应的变量和方法；`ref()`相当于`state`，`computed()`相当于`getters`，`function()`相当于`actions`。



**代码演示：**

```js
// \src\store\useStore.js
import { defineStore } from 'pinia'

// 选项式写法（对象）
export const useStore = defineStore('main', {
    state : ()=>({
        
    }),
    getters : {},
    actions : {}
});

// 组合式写法（函数）
export const useStore = defineStore('main', ()=> {
    // ref() 定义的数据
    // compouted() 定义的计算属性
    // function() 定义的方法
});
```



### 8.3：`state`

> `state`是`store`的核心部分，主要存储的是共享数据



#### 8.3.1：选项式API

> 1. `store`采用的选项式API时，`state`选项为函数返回的对象，在其定义共享的数据
>
> 2. 在其他页面使用时，遵循以下步骤：
>
>    1. 引入对应的`store`文件
>
>    2. 在`pinia`中引入`mapState`或者`mapWritableState`，这两个辅助函数的用法和作用相同，都是用来将`store`的状态映射到组件的计算属性中
>
>       - `参数一`：模块化的`store`文件
>       - `参数二`：字符串数组或对象
>
>    3. `mapState`函数映射的计算属性是只读的；
>
>       `mapWritableState`函数映射的计算属性可以更改
>
>    4. 这俩映射的都是响应式的



**代码演示：**

```vue
<script>
    import { mapWritableState } from 'pinia'
    import { useStore } from '@/store/useStore'

    export default {
        computed : {
            ...mapWritableState(useStore, {
                name : 'name',
                age : 'age'
            })
        }
    }
</script>
<template>
    用户名：<input type="text" v-model="name"><br>
    年龄：<input type="number" v-model="age"><br>
</template>
```




#### 8.3.2：组合式API

> 1. `store`采用的是组合式API时，使用`ref`定义的数据即为响应式数据，最后要把要暴露的数据`return`出去。
> 2. 在组合式API组件中，直接引入对应的`store`，通过`store`对象直接获取和修改`state`。
> 3. 如果想在组件中自定义变量来接收`store`中的`state`中共享的数据，我们可以这样做：
>    1. 使用`computed(()=> store.dataName)`，具有响应式，但不能i修改，是只读格式
>    2. 使用`storeToRefs(store)`从`store`解构想要的`state`，具有响应式，可直接修改，可自定义名称



**代码演示：**

```vue
<script setup>
import { useUserStore } from '@/store/useUserStore';
import { computed } from 'vue';
import { storeToRefs } from 'pinia';

const userStore = useUserStore();

const { 
    name: u_name,
    age : u_age,
    height : u_height,
    weight : u_weight
} = storeToRefs(userStore);

let c_name = computed(() => userStore.name);
let c_age = computed(() => userStore.age);
let c_height = computed(() => userStore.height);
let c_weight = computed(() => userStore.weight);
</script>
<template>
    <h2>从store直接获取：</h2>
    <ul>
        <li>姓名：{{ userStore.name }}</li>
        <li>年龄：{{ userStore.age }}</li>
        <li>身高：{{ userStore.height }}</li>
        <li>体重：{{ userStore.weight }}</li>
    </ul>
    <div>
        <button @click="userStore.height++">身高++</button>
    </div>
    <hr>
    <h2>computed方式获取：（只读）</h2>
    <ul>
        <li>姓名：{{ c_name }}</li>
        <li>年龄：{{ c_age }}</li>
        <li>身高：{{ c_height }}</li>
        <li>体重：{{ c_weight }}</li>
    </ul>
    <div>
        <button @click="c_age++">身高++</button>
    </div>
    <hr>
    <h2>storeToRef方式获取：</h2>
    <ul>
        <li>姓名：{{ u_name }}</li>
        <li>年龄：{{ u_age }}</li>
        <li>身高：{{ u_height }}</li>
        <li>体重：{{ u_weight }}</li>
    </ul>
</template>
```



### 8.4：`getters`

> `getters`是计算得到的新的共享数据，当依赖的数据发生变化时重新计算，所以其他组件包括`store`自己不要对其修改



#### 8.4.1：定义`Getter`



##### 8.4.1.1：选项式API

> 1. `store`采用的是选项式模式时，`getters`选项中声明的函数名即为计算属性
>    1. 在其函数内可通过`this`关键字来获取`store`实例，也可在其方法参数上设置第一个参数来指向`store`。
>    2. 如果采用的是箭头函数的话，无法使用`this`关键字，为了更方便使用`store`实例，可为其箭头函数设置第一个参 数来指向`store`。



**代码演示：**

```vue
<!-- user.vue -->
<script>
    import { mapWritableState } from 'pinia';
    import {useUserStore} from '@/store/useUserStore';
    export default {
        computed : {
            ...mapWritableState(useUserStore, ['name', 'say', 'ageState']),
        }
    }
</script>
<template>
<h1>{{name}}</h1>
    <h2>{{ say }}，还{{ ageState }}</h2>
</template>
```

```js
//useUserStore.js
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user-store', {
    state : ()=>({
        name : 'jiale',
        age : 20
    }),
    getters : {
        say() {
            return `我是${this.name}，我今年${this.age}岁了`;
        },
        ageState : store=> {
            if(store.age >= 18) {
                return '已成年';
            } else {
                return '未成年';
            }
        }
    }
});
```



##### 8.4.1.2：组合式API

> 1. `store`采用的组合模式时，可以通过`computed()`函数通过计算得到新的数值，再将其`return`暴露出去。
> 2. 在组合式API组件中，访问`store`中的`getters`和`state`类似，直接引入对应的`store`，通过`store`对象直接获取`getters`，但是如果对其修改则会报错。<font color=red>（也可以使用`computed`方式和`storeToRefs()`辅助函数）</font>



### 8.5：`actions`

> 1. `store`采用的是选项式模式时，`actions`选项中声明的函数即可共享其函数，在其函数内可通过`this`来获取整个`store`实例。
> 2. 组件中访问`Actions`（选项式）
>    1. 可以使用`mapActions(storeObg, array | object)`帮助器将`actions`映射为当前组件的函数
>       - `storeOBJ`：引入的`store`对象
>       - `array | object`：字符串数组或者对象形式
> 3. `store`采用组合模式时，可先声明函数，再将其`return`暴露出去即可共享其函数
> 4. 组件中访问`Actions`（组合式）
>    1. 可以直接将`store`引入，通过`store`对象直接获取`actions`。
>    2. 如果想将`store`中的`actions`中函数映射为本地组件的函数，可将`store`解构出对应的函数即可，也可以自定义函数名，这里不能用`storeToRefs(store)`函数



**代码演示：**

```vue
<!-- user.vue -->

<script setup>
    // import { mapWritableState, mapActions } from 'pinia';
    // import { useUserStore } from '@/store/useUserStore';

    // export default {
    //     computed : {
    //         ...mapWritableState(useUserStore, ['student']),
    //         ...mapWritableState(useUserStore, {
    //             say : 'say'
    //         })
    //     },
    //     methods : {
    //         ...mapActions(useUserStore, {
    //             setName : 'setStudentName'
    //         })
    //     }
    // }
    import { storeToRefs } from 'pinia';
    import { useUserStore } from '@/store/useUserStore'

    let userStore = useUserStore();
    const {
        student : stuInfo,
        say,
        setStudentName : setname
    } = storeToRefs(userStore);
    const {
        setStudentName : setName
    } = userStore;
</script>

<template>
    <h2>学生：</h2>
    <ul>
        <li>{{ stuInfo }}</li>
        <li>{{ say }}</li>
    </ul>
    <button @click="setName('bao')">修改名字</button>
</template>
```



```js
// useUserStore.js

import { defineStore } from 'pinia';
import { reactive, computed } from 'vue';

export const useUserStore = defineStore('userStore', ()=> {
    const student = reactive({
        name : 'jiale',
        age : 20
    });
    let say = computed(()=> `我叫${student.name}，我今年${student.age}岁了。`);
    function setStudentName(n) {
        student.name = n;
    }

    return {
        student,
        say,
        setStudentName
    }
});

// {
//     state: () => ({
//         student: {
//             name: 'jiale',
//             age: 20
//         }
//     }),
//         getters : {
//         say() {
//             return `我叫${this.student.name}，我今年${this.student.age}岁了。`;
//         }
//     },
//     actions: {
//         setStudentName(n) {
//             this.student.name = n;
//         }
//     }
// }
```

