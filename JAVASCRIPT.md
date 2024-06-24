# Javascript coding guidelines

There are a number of style guides out there. Here a few important things that we should follow:

- [1. Linting / Auto-formatting](#1-linting--auto-formatting)
  - [Formatting with Prettier](#formatting-with-prettier)
  - [Linting with Eslint](#linting-with-eslint)
    - [Ignore linting warning/refactoring/convention](#ignore-linting-warningrefactoringconvention)
- [2. Naming conventions](#2-naming-conventions)
- [3. Comments and JSDoc](#3-comments-and-jsdoc)
- [4. Single statement](#4-single-statement)
- [5. Vue 3 Best Practice](#5-vue-3-best-practice)
  - [Styling](#styling)
  - [Component data](#component-data)
  - [Component method](#component-method)
    - [Definition](#definition)
    - [Method called from template](#method-called-from-template)
- [Debugging on Mobile Phone](#debugging-on-mobile-phone)

**The foremost goal is that reading and understanding your javascript code is easy for someone else (or yourself in a few months time).**

## 1. Linting / Auto-formatting

### Formatting with Prettier

For our javascript projects we are using [Prettier](https://prettier.io/) to automatically format your code. We highly recommend to configure you IDE to format your code using Prettier on Save. Here below is a Prettier configuration example

```yaml
printWidth: 100
singleQuote: true
semi: false
trailingComma: es5
tabWidth: 4
jsxSingleQuote: false
jsdocParser: true
jsdocDescriptionWithDot: true
jsdocVerticalAlignment: true
overrides:
    - files: '*.md'
      options:
          tabWidth: 2
          proseWrap: 'always'

```

### Linting with Eslint

Although formatting is good, it doesn't check for syntax errors or bad code practice.
Therefore we also use a linter; [Eslint](https://eslint.org/)

Usually we should have such ESlint configuration:

```json
{
    "env": {
        "es2021": true,
        "node": true
    },
    "plugins": ["prettier"],
    "extends": [
        "eslint:recommended", 
        "plugin:vue/essential",
        "prettier",
        "plugin:vue/recommended",
        "plugin:cypress/recommended",
        "plugin:prettier-vue/recommended",
    ],
    "parserOptions": {
        "ecmaVersion": 12,
        "parser": "babel-eslint",
    },
    "rules": {
        "camelcase": ["error", { "properties": "never" }],
        "curly": "error",
        "max-len": ["warn", { "code": 100 }],
        "no-unused-vars": "warn"
    }
}
```

#### Ignore linting warning/refactoring/convention

There might be good reason to disable locally some linting messages. When doing this, the reason why we disable a rule
should be documented next to the disable pragma.

For more detail in ignoring eslint issues see
[ESLint Disabling Rules](https://eslint.org/docs/user-guide/configuring/rules#disabling-rules)

## 2. Naming conventions

Javascript code must follow these naming conventions:

- file name: camelCase
- constant: UPPER_CASE
- variable: camelCase
- function/method: camelCase
- argument: camelCase
- class: PascalCase

## 3. Comments and JSDoc

Code is documented using [JSDoc](https://jsdoc.app/). Make sure to document every public class and functions.

## 4. Single statement

ALWAYS enclosed with curly brace single if/else/while/do while/for single statements

:x: **Incorrect**

```javascript
if (test) return

if (test)
    doSomething()

while(1)
    doSomething()

for (let i=0; i < test; i++) doSomething()
```

:white_check_mark: **Correct**

```javascript
if (test) {
    return
}

if (test) {
    doSomething()
}

while(1) {
    doSomething()
}

for (let i=0; i < test; i++) {
    doSomething()
} 
```

## 5. Vue 3 Best Practice

See first [Vue 3 Best Practice](https://vuejs.org/v2/style-guide/)

### Styling

- Always use directive shorthands; `:` for `v-bind`, `@` for `v-on`, `#` for `v-slot`
- Always use Single-file component whenever possible
- Single-file component must always have the following element order

    ```html
    <script setup>/* ... */</script>
    <template>...</template>
    <style>/* ... */</style>
    ```

- Use indentation within `<template></template>` tag but not on `<script></script>` and `<style></style>` tags

    ```html
    <script setup>
    const myString = '<script></script> tag omit the first indentation level'
    const isScript = true

    if (isScript) {
        console.log(myString)
    }
    </script>

    <template>
        <div class="my-template">
            <p>My HTML template is fully indented</p>
        </div>
    </template>

    <style scoped>
    .my-template {
        background-color: green;
    }
    </style>
    ```

- Avoid using element selector with `scoped` (see [Vue 3 Best Practice - Element selectors with `scoped`](https://vuejs.org/v2/style-guide/#Element-selectors-with-scoped-use-with-caution)), but use `scoped` class or id selector instead.

    ```html
    <!-- BAD -->
    <template>
        <button>X</button>
    </template>

    <style scoped>
    button {
        background-color: red;
    }
    </style>
    ```

    ```html
    <!-- GOOD -->
    <template>
        <button class="btn btn-close">X</button>
    </template>

    <style scoped>
    .btn-close {
        background-color: red;
    }
    </style>
    ```

- Always add the `*.vue` extension in import of Vue single file component. This is required by some IDE (Vscode/Volar) and omitting the extension has been deprecated by Vue 3.

    :x: **Incorrect**

    ```javascript
    import MyComponent from "@/components/MyComponent"
    ```

    :white_check_mark: **Correct**

    ```javascript
    import MyComponent from "@/components/MyComponent.vue"
    ```

### Component data

Don't use complex object (e.g. instance from an external library) in the `data` section of a component (Option API) or wrapped in a `ref` (Composition API). Refs and the data section are made deeply reactive which means that each object is recursively analysed by Vue and each object properties are proxied. This would have a negative impact on performance and might break the complex object.

:x: BAD

```javascript
// BAD
import { Map } from 'ol'

export default {
    data(): {
        return {
            myBoolean: true
            myMap: new Map({ controls: [] })
        }
    },
}
```
```javascript
import { ref } from 'vue'
import { Map } from 'ol'

const myBoolean = ref(true)
const myMap = ref(new Map({ controls: [] }))
```

:white_check_mark: GOOD

```javascript
// GOOD
import { Map } from 'ol'

export default {
    data(): {
        return {
            myBoolean: true
        }
    },
    created(): {
        this.map = new Map({ controls: [] })
    },
}
```
```javascript
import { ref } from 'vue'
import { Map } from 'ol'

const myBoolean = ref(true)
const myMap = new Map({ controls: [] })
```


### Component method

#### Definition

Don't use arrow function to define Option API component method (arrow function prevent `Vue` from binding the appropriate `this` value, see [Vue 3 - Methods](https://v3.vuejs.org/guide/data-methods.html#methods))

:x: BAD

```javascript
// BAD

const app = Vue.createApp({
    data() {
        return {count: 0}
    },
    methods: {
        increment: () => {
            // `this` don't necessarily refer to the component depending on the context
            this.count++
        }
    }
})
```

:white_check_mark: GOOD

```javascript
// GOOD

const app = Vue.createApp({
    data() {
        return {count: 0}
    },
    methods: {
        increment() {
            // `this` always refer to the component
            this.count++
        }
    }
})
```

Also prefer the short syntax `increment() {}` to `increment: function() {}`

#### Method called from template

Methods called from a template (except for event listeners) should not have any side effects, such as changing data or triggering asynchronous processes. If you find yourself tempted to do that you should probably use a [lifecycle hook](https://v3.vuejs.org/guide/instance.html#lifecycle-hooks) instead. For more infos about this see [Vue 3 - Methods](https://v3.vuejs.org/guide/data-methods.html#methods).

## Debugging on Mobile Phone

Debugging issue that only touch Mobile Phones, especially IPhones, can be quite difficult without a Mac.
For this task you can use [inspect.dev](https://inspect.dev) which works quite well.

This tool requires a subscriptions for troubleshooting on IOS devices, currently the following user
have a yearly subscription:

- Brice Schaffner (private subscription)
