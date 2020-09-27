---
title: Vue3 Composition API
date: 2020-09-25 16:28:29
categories:
  - 前端
tags:
  - Vue
  - Vue3
---

为了能够提取`Vue`组件中可复用的逻辑和代码块，解决在大型应用中代码的共享问题，`Vue3`提供了全新的`Composition API`。

<!-- more -->

在多数情况下使用组件的选项`(data, computed, methods, watch)`来组织逻辑是可行的，但是当组件变得更大更复杂时，**逻辑相关**的数量也会增加，这会使得在不同组件选项中相关逻辑变得相关交叉切复杂，难以阅读和理解，`data, computed, methods, watch`之间都有着相关关联的逻辑，`Composition API`就是为了将逻辑相关的代码进行抽离，使其成为可共享的代码片段。

```js
// src/components/UserRepositories.vue
// 1 2 3 分别表示三个不通过的逻辑和其在不同组件选项中的关联
// 1 获取 repositories，并在 user 变化是刷新
// 2 查询 repositories
// 3 过滤 repositories
export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { type: String }
  },
  data () {
    return {
      repositories: [], // 1
      filters: { ... }, // 3
      searchQuery: '' // 2
    }
  },
  computed: {
    filteredRepositories () { ... }, // 3
    repositoriesMatchingSearchQuery () { ... }, // 2
  },
  watch: {
    user: 'getUserRepositories' // 1
  },
  methods: {
    getUserRepositories () {
      // using `this.user` to fetch user repositories
    }, // 1
    updateFilters () { ... }, // 3
  },
  mounted () {
    this.getUserRepositories() // 1
  }
}
```

## `setup`组件选项

### 执行时期

在组件`created`**之前**执行，一旦`props`解析完成，它将作为组件 API 的入口点。

**注意：** 由于当`setup`执行时组件并未有创建，所以`setup`内部不能获取`this`，这就意味着除了`props`以外，所有在组件内声明的属性如：`local state, computed, properties, methods`都不无法获得。

现在使用`setup`剥离上述的逻辑 1

```js
// src/components/UserRepositories.vue
import { fetchUserRepositories } from "@/api/repositories";
export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: { user: { type: String } },
  setup(props) {
    let repositories = [];
    const getUserRepositories = async () => {
      repositories = await fetchUserRepositories(props.user);
    };
    return {
      repositories,
      getUserRepositories,
    };
  },
};
```

此时`repositories`还无法正常工作，因为它不是响应式的，无法改变它的值，接下来将通过`ref`修复它。

### 通过`ref`使变量变为响应式

`ref`接收一个参数，并返回一个包装后的带有`value`属性的对象，也直接使用或者改变变量的值。

```js
import { ref } from "vue";
const counter = ref(0);
console.log(counter); // {value: 0}
console.log(counter.value); // 0

counter.value++;
console.log(counter.value); // 1
```

之所以采用一个对象对值进行包装的原因是为了不同数据类型之间的行为统一，`Number, String`属于值类型，无法做到在整个应用中的响应，而包装成对象的引用类型则解决这一问题。

通过`ref`上述的代码就可以修改为如下的形式：

```js
import { fetchUserRepositories } from "@/api/repositories";
import { ref } from "vue";
export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { type: String }
  },
  setup(props) {
    const repositories = ref([])
    const getUserRepositories = async () => {
      repositories.value = await fetchUserRepositories(props.user)
    }

    return {
      repositories,
      getUserRepositories
    }
  },
  data () {
    return {
      filters: { ... }, // 3
      searchQuery: '' // 2
    }
  },
  computed: {
    filteredRepositories () { ... }, // 3
    repositoriesMatchingSearchQuery () { ... }, // 2
  },
  watch: {
    user: 'getUserRepositories' // 1
  },
  methods: {
    getUserRepositories () {
      // using `this.user` to fetch user repositories
    }, // 1
    updateFilters () { ... }, // 3
  },
  mounted () {
    this.getUserRepositories() // 1
  }
}
```

### 向`setup`中加入生命周期

composition API 拥有和声明周期中相同的函数，但是会加上`on`的前缀，如`mounted -> onMounted`。这些函数接受一个回调函数，当声明周期的 hook 被调用时会被执行。

```js
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted } from 'vue'

// in our component
setup (props) {
  const repositories = ref([])
  const getUserRepositories = async () => {
    repositories.value = await fetchUserRepositories(props.user)
  }

  onMounted(getUserRepositories) // on `mounted` call `getUserRepositories`

  return {
    repositories,
    getUserRepositories
  }
}
```

### 使用`watch`监听变化

`watch`接受三个参数

- 需要监听响应式的引用或者`getter`方法
- 回调函数
- 可选配置

```js
import { ref, watch } from "vue";

const counter = ref(0);
watch(counter, (newValue, oldValue) => {
  console.log("The new counter value is: " + newValue);
});
```

因此我们最初的例子可以改造为如下形式，这样我们就将第一种的逻辑相关部分的代码统一到一处。

```js
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch, toRefs } from 'vue'

// in our component
setup (props) {
  // using `toRefs` to create a Reactive Reference to the `user` property of props
  const { user } = toRefs(props)

  const repositories = ref([])
  const getUserRepositories = async () => {
    // update `props.user` to `user.value` to access the Reference value
    repositories.value = await fetchUserRepositories(user.value)
  }

  onMounted(getUserRepositories)

  // set a watcher on the Reactive Reference to user prop
  watch(user, getUserRepositories)

  return {
    repositories,
    getUserRepositories
  }
}
```

### 独立的`computed`属性

类似于`ref watch`，`computed`同样可以在组件外创建。

```js
import { ref, computed } from "vue";

const counter = ref(0);
const twiceTheCounter = computed(() => counter.value * 2);

counter.value++;
console.log(counter.value); // 1
console.log(twiceTheCounter.value); // 2
```

`computed`函数返回一个只读的响应式引用，我们要像`ref`一样使用`.value`来访问计算属性的值。

### 使用`composition function`封装独立的逻辑相关代码

逻辑模块 1

```js
// src/composables/useUserRepositories.js

import { fetchUserRepositories } from "@/api/repositories";
import { ref, onMounted, watch } from "vue";

export default function useUserRepositories(user) {
  const repositories = ref([]);
  const getUserRepositories = async () => {
    repositories.value = await fetchUserRepositories(user.value);
  };

  onMounted(getUserRepositories);
  watch(user, getUserRepositories);

  return {
    repositories,
    getUserRepositories,
  };
}
```

逻辑模块 2

```js
// src/composables/useRepositoryNameSearch.js

import { ref, computed } from "vue";

export default function useRepositoryNameSearch(repositories) {
  const searchQuery = ref("");
  const repositoriesMatchingSearchQuery = computed(() => {
    return repositories.value.filter((repository) => {
      return repository.name.includes(searchQuery.value);
    });
  });

  return {
    searchQuery,
    repositoriesMatchingSearchQuery,
  };
}
```

逻辑模块 3

```js
// src/composables/useRepositoryFilters.js

import { ref, computed } from "vue";

export default function useRepositoryFilters(repositories) {
  const filters = ref({});
  const updateFilters = (newFilter) => {
    filters.value = newFilter;
  };
  const filteredRepositories = computed(() => {
    return repositories.value.filter((repository) => {
      for (let key in filter.value) {
        if (repository[key] !== filter.value[key]) {
          return false;
        }
      }
      return true;
    });
  });
  return {
    filters,
    updateFilters,
    filteredRepositories,
  };
}
```

最后重构后的结果：

```js
// src/components/UserRepositories.vue
import { toRefs } from "vue";
import useUserRepositories from "@/composables/useUserRepositories";
import useRepositoryNameSearch from "@/composables/useRepositoryNameSearch";
import useRepositoryFilters from "@/composables/useRepositoryFilters";

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { type: String },
  },
  setup(props) {
    const { user } = toRefs(props);
    // 逻辑相关1
    const { repositories, getUserRepositories } = useUserRepositories(user);
    // 逻辑相关2
    const {
      searchQuery,
      repositoriesMatchingSearchQuery,
    } = useRepositoryNameSearch(repositories);
    // 逻辑相关3
    const {
      filters,
      updateFilters,
      filteredRepositories,
    } = useRepositoryFilters(repositoriesMatchingSearchQuery);

    return {
      // Since we don’t really care about the unfiltered repositories
      // we can expose the end results under the `repositories` name
      repositories: filteredRepositories,
      getUserRepositories,
      searchQuery,
      filters,
      updateFilters,
    };
  },
};
```
