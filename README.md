# vue-fade-scrollbar
Wrapper component that replaces the element's own scrollbar. The new scrollbar can auto-hide on various media queries, appear on hover content and doesn't affect the content.
[See here the Demo](https://kirillidze.github.io/vue-fade-scrollbar/)

NPM package: https://www.npmjs.com/package/vue-fade-scrollbar

## Installation

```bash
npm install --save vue-fade-scrollbar
```

or

```bash
yarn add vue-fade-scrollbar
```
## Import

### Global Registration

```javascript
import Vue from 'vue'
import VueFadeScrollbar from "./VueFadeScrollbar"

Vue.component('vue-fade-scrollbar', VueFadeScrollbar)
```

### In Component Registration:

```html
<template>
  ...
</template>
<script>
import VueFadeScrollbar from 'vue-fade-scrollbar'

export default {
  name: 'ComponentName',
  components: {
    VueFadeScrollbar,
    ...
  },
  ...
}
</script>
```

### Not Using Webpack

If you don't use webpack (for example in CodePen etc):

```html
<link rel="stylesheet" href="node-modules/vue-fade-scrollbar/distvue-fade-scrollbar.css" />
<script src="node-modules/vue-fade-scrollbar/dist/vue-fade-scrollbar.umd.js"></script>
```

and then you still need to register it:

```javascript
Vue.component('vue-fade-scrollbar', VueFadeScrollbar)
```

## Basic Usage

```html
<template>
  <VueFadeScrollbar :visible="mediaQuery">
    ...
  </VueFadeScrollbar>
</template>

<script>
    export default {
        data: {
            mediaQuery: 'm-down',
        },
    }
</script>
```
### Props

- `visible` (default: `''`): media queries for scrollbar visibility

    | name       | value                      |
    | ---------- | -------------------------- |
    | xs         | min-width: 375px           | 
    | s          | min-width: 480px           | 
    | m          | min-width: 768px           | 
    | l          | min-width: 1024px          | 
    | xl         | min-width: 1440px          | 
    | xs-down    | max-width: 374px           | 
    | s-down     | max-width: 479px           | 
    | m-down     | max-width: 767px           | 
    | l-down     | max-width: 1023px          | 
    | xl-down    | min-width: 1439px          | 
    | xl-down    | max-width: 1439px          |

- `thumb` (type: `Object`). Thumb properties:

  | property    | type      | default   | description                         | 
  | ----------- | --------- | --------- | ----------------------------------- |
  | thikness    | string    | 10px      | thumb width                         |
  | left        | string    | 1px       | thumb left indent inside the track  |
  | right       | string    | 1px       | thumb right indent inside the track |
  | radius      | string    | 0         | thumb border-radius                 |
  | color       | string    | #c7ccd1   | thumb color                         |
  | hoverColor  | string    | #8f99a3   | thumb color on hover                |
  | activeColor | string    | #666666   | thumb active color                  |

- `thumbClasses` (type: `String, Array, Object`, default: `''`): custom classes for thumb
- `trackColor` (type: `String`, default: `'transparent'`)
