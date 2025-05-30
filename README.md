vue2-teleport-polyfill
-------------

Vue2-teleport-polyfill forked and modified from [Mechazawa/vue2-teleport](https://github.com/Mechazawa/vue2-teleport)

with support for passing an `HTMLElement` as the `:to` prop.

---

[![npm version](https://badge.fury.io/js/vue2-teleport.svg)](https://badge.fury.io/js/vue2-teleport)

This package is an alternative to [vue3's teleport component][oc-teleport]. You can use the documentation provided by vue as a starting point to using this package.

## Installation
```sh
npm install vue2-teleport-polyfill
```

## Example

```vue
<template>
  <div>
      <button @click="modalOpen = true">
          Open full screen modal! (With teleport!)
      </button>
    
      <Teleport to="body">
        <div v-if="modalOpen" class="modal">
          <div>
            I'm a teleported modal! 
            (My parent is "body")
            <button @click="modalOpen = false">
              Close
            </button>
          </div>
        </div>
      </Teleport>
  </div>
</template>

<script>
import Teleport from 'vue2-teleport-polyfill';

export default {
  components: {
    Teleport,
  },
  data() {
    return { 
      modalOpen: false
    }
  }
}
</script>
```

[Local demo](http://localhost:4321/examples/basic.html)

```bash
npm run dev
```

## License

This project is licensed under the [CC0-1.0 License][license].


[license]: /LICENSE
[oc-teleport]: https://vuejs.org/guide/built-ins/teleport.html
