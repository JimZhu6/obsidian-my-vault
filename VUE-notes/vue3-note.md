# Vue3 笔记

> [预览版仓库](https://github.com/vuejs/vue-next-webpack-preview)、[Vue3仓库](https://github.com/vuejs/vue-next)



## 新特性

### Composition API

在 Vue3 中，可直接通过引入的方式执行 Vue2 中的各种 Function，代码结构更加精简。

```vue
<template>
  <div class="counter">
    <p>count: {{ count }}</p>
    <p>NewVal (count + 2): {{ countDouble }}</p>
    <button @click="inc">Increment</button>
    <button @click="dec">Decrement</button>
    <p> Message: {{ msg }} </p>
    <button @click="changeMessage()">Change Message</button>
  </div>
</template>
<script>
import { ref, computed, watch } from 'vue'
export default {
  setup() {
    /* ---------------------------------------------------- */
    let count = ref(0)
    const countDouble = computed(() => count.value * 2)
    watch(count, newVal => {
      console.log('count changed', newVal)
    })
    const inc = () => {
      count.value += 1
    }
    const dec = () => {
      if (count.value !== 0) {
        count.value -= 1
      }
    }
    /* ---------------------------------------------------- */
    let msg= ref('some text')
    watch(msg, newVal => {
      console.log('msg changed', newVal)
    })
    const changeMessage = () => {
      msg.value = "new Message"
    }
    /* ---------------------------------------------------- */
    return {
      count,
      inc,
      dec,
      countDouble,
      msg,
      changeMessage
    }
  }
}
</script>
```

我们同样可以抽取出重复功能的代码:

```js
// message.js
import { ref, watch } from "vue";
export function message() {
  let msg = ref(123);
  watch(msg, (newVal) => {
    console.log("msg changed", newVal);
  });
  const changeMessage = () => {
    msg.value = "new Message";
  };
  return { msg, changeMessage };
}
```

在其他组件中使用上面组件:

```vue
<template>
  <div class="counter">
    <p>count: {{ count }}</p>
    <p>NewVal (count + 2): {{ countDouble }}</p>
    <button @click="inc">Increment</button>
    <button @click="dec">Decrement</button>
    <p>Message: {{ msg }}</p>
    <button @click="changeMessage()">change message</button>
  </div>
</template>
<script>
import { ref, computed, watch } from 'vue'
import { message } from './common/message'
export default {
  setup() {
    let count = ref(0)
    const countDouble = computed(() => count.value * 2)
    watch(count, newVal => {
      console.log('count changed', newVal)
    })
    const inc = () => {
      count.value += 1
    }
    const dec = () => {
      if (count.value !== 0) {
        count.value -= 1
      }
    }
    let { msg, changeMessage } = message()
    return {
      count,
      msg,
      changeMessage,
      inc,
      dec,
      countDouble
    }
  }
}
</script>
```



### Multiple root elements

在 Vue3 中，可以直接在`<template></template>`中使用任意数量的标签:

```vue
<template>
  <p> Count: {{ count }} </p>
  <button @click="increment"> Increment </button>
  <button @click="decrement"> Decrement </button>
</template>
```



### Suspense

Suspense 是一个 Vue 3 新特性。

通常前后端交互是一个异步的过程： 默认我们提供一个加载中的动画，当数据返回时配合使用 `v-if` 来控制数据显示。

Suspense 的出现大大简化了这个过程：它提供了 `default` 和 `fallback` 两种状态：

```vue
<template>
  <Suspense>
    <template #default>
      <div v-for="item in articleList" :key="item.id">
        <article>
          <h2>{{ item.title }}</h2>
          <p>{{ item.body }}</p>
        </article>
      </div>
    </template>
    <template #fallback>
      Articles loading...
    </template>
  </Suspense>
</template>
<script>
import axios from 'axios'
export default {
  async setup() {
    let articleList = await axios
      .get('https://jsonplaceholder.typicode.com/posts')
      .then(response => {
        console.log(response)
        return response.data
      })
    return {
      articleList
    }
  }
}
</script>
```



### Multiple v-models

在 Vue 3 中，我们可以将任意数量的 v-model 绑定到我们的定制组件上:

```vue
<template>
  <survey-form v-model:name="name" v-model:age="age">
    {" "}
  </survey-form>
</template>
```

```vue
<template>
    <div>
        <label>Name: </label>
        <input :value="name" @input="updateName($event.target.value)" />
        <label>Age: </label>
        <input :value="age" @input="updateAge($event.target.value)" />
    </div>
</template>
<script>
    export default {
      props: {
        name: String,
        age: Number
      },
      setup(props, { emit }) {
        const updateName = value => {
          emit('update:name', value)
        }
        const updateAge = value => {
          emit('update:age', +value)
        }
        return { updateName, updateAge }
      }
    }
</script>
```



### Remove Filter

Vue 3 抛弃了 `Filter` 的用法，更推荐使用计算属性或方法来实现：

```vue
<!-- vue 2.x -->
{{ date | format }}

<!-- vue 3.0 -->
{{ format(date) }}
```



