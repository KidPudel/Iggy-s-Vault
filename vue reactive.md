State that can trigger updates when changing is **reactive**

To declare reactive state we can use `reactive()` API, but it only works *on objects*
```js
import { reactive } from 'vue'

const counter = reactive({
  count: 0
})

console.log(counter.count) // 0
counter.count++
```
`ref()` comes to play. It can take any value type and create an object exposes inner state under a `.value` property
```js
import { ref } from 'vue'

const message = ref('Hello World!')

console.log(message.value) // "Hello World!"
message.value = 'Changed'
```

