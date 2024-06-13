directives are prefixed with `v-` to indicate that they are special attributes provided by the Vue. They apply special reactive behavior to manipulate/update rendered DOM
"keep this element's inner HTML up-to-date with the `rawHtml` property on the current active instance."

- `v-html`: allows to parse HTML into an element utilizing the properties
- `v-bind`: *bind attribute* to an expression in a tag (shorthand is `:`)
- `v-if`: conditionally render an element
- `v-for`: render a list of items
- `v-on`: Attach an event listener (shorthand is `@`)

```html

<script setup>
const count = ref(0)
function increment() {
	count.value++
}
</script>

<template>
<!--@click="count++" -->
<button v-on:click="increment">count {{count}}</button>
</template>

```

```html
<script setup>
import { ref } from 'vue'

const text = ref('')

function onInput(e) {
  text.value = e.target.value
}
</script>

<template>
  <input :value="text" @input="onInput" placeholder="Type here">
  <p>{{ text }}</p>
</template>
```