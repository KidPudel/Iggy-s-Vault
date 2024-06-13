every vue application starts with `createApp` function, to which we pass a **root** component, which is necessary to put children in
```js
import { createApp } from 'vue'
import App from './App/vue'

const app = createApp(App)
```
When using [[sfc]] we typically import that file as a root component

like in flutter app consists of component tree

```
App (root component)
├─ TodoList
│  └─ TodoItem
│     ├─ TodoDeleteButton
│     └─ TodoEditButton
└─ TodoFooter
   ├─ TodoClearButton
   └─ TodoStatistics
```

But nothing would be rendered until we *mount* application into an *actual* [[DOM]] element or a selector string

```html
<div id="app"></div>
```
```js
app.mount("#app")
```

This will render an app inside a container


# installation
install and create vue project
```
$ npm create vue@latest
```
start dev server
```
cd <your-project-name>
npm install
npm run dev
```
ship application to the server
```
$ npm run build
```
# in-DOM root component template
The template for the root component is usually a part of the component itself (like `App`), but we can define the root component template *separately* by writing it directly *inside the mount container*

```js
import {createApp} from 'vue'
const app = createApp({
	data() {
		return {
			count: 0
		}
	}
})
app.mount("#app")
```
```html
<div id="app">
	<button @click="count++">{{count}}</button>
</div>
```

Vue will automatically use `innerHTML` as a template if the root component does not already have a `tample` option


# App configuration
application instance exposes `.config` object that allows for configuring some of app-level options
- error handling `app.config.errorHandler = (err) => { /* handle error */}`

## Register components
Also we can register components globally, to then access it anywhere we want
```js
app.component('my-component', {
	// ...
})

const myComponent = app.component('my-component')
```


# Multiple application instances
`createApp` not limiting to one application, we can create multiple applications with its own scope for configuration and global assets, **on the same page!**

```js
const app = createApp({})
app.mount('#container1')
const app2 = createApp({})
app2.mount('#container2')
```



# template syntax

Vue uses HTML based syntax that allows you to declaratively bind rendered DOM to the underlying component instance's data.
## Text interpolation
```html
<span>Message: {{message}}</span>
```
Vue threats data in in double mustaches as a plain text. To interpret as an HTML - use `v-html` [[Vue directive]] attribute
```html
<p>Using text interpolation: {{ rawHtml }}</p> 
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```
Using text interpolation: This should be red.
Using v-html directive: <span style="color: red">This should be red.</span>


## Attribute binding
To dynamically *bind HTML attributes* to tags use `v-bind` to change the look and behavior
```html
<div v-bind:id="dynamicId"></div>
<div :id="dynamicId"></div>
<div :id></div> // if name is the id
```
`v-bind` instructs Vue to keep element's `id` attribute in sync with component's property

### Boolean attributes
Attributes to indicate true/false values by their presence in the element, that are also are bounded via `v-bind`
```html
<button :disabled="isButtonDisabled">Click</button>
```
`isButtonDisabled` will be ***only** included if it has true value, or value is empty string*

### Dynamically bind multiple attributes
```js
const objectOfAttrs = { 
	id: 'container', class: 'wrapper', style: 'background-color:green' 
}
```
```html
<div v-bind="objectOfAttrs"></div>
```


# Full power of JavaScript expressions
We can use whole JS inside data binding, but i **can have only one expression**
```js
{{message + 1}}
{{message.split('')}}
```
but it can have **only expressions, not statements**

# Defile components
To define components in a script we can use `export default` 
like that:
```vue
<template>
  <div id="desk">
    <h1>Chinese Bee Dictation</h1>
    <button v-on:click="increment">count{{ count }}</button>
    <vue-drawing-canvas ref="VueCanvasDrawing" :width="500" />
    
  </div>
</template>

<script>
import VueDrawingCanvas from "vue-drawing-canvas"
import {ref} from 'vue'

export default {
  name: "App",
  components: {
    VueDrawingCanvas,
  },
  // hook
  setup() {
    const count = ref(0)
    function increment() {
      count.value++
    }

    return {
      count,
      increment
    }
    
  }
}
</script>
```

or we can use