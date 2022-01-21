---
title: 'Modules in Vue 3'
metaTitle: 'Modules in Vue 3 - Guide - Handsontable Documentation'
permalink: /11.1/vue3-modules
canonicalUrl: /vue3-modules
---

# Modules in Vue 3

[[toc]]

To reduce Handsontable's impact on the size of your Vue 3 app, import only the [modules](@/guides/building-and-testing/modules.md) that you actually use.

[Find out which Vue 3 versions are supported &#8594;](@/guides/integrate-with-vue3/vue3-installation.md#vue-3-version-support)

## Using modules with Vue 3

To use modules with Handsontable's [Vue 3 wrapper](@/guides/integrate-with-vue3/vue3-installation.md), follow the steps below:

### Step 1: Import required modules

No matter which [optional modules](@/guides/building-and-testing/modules.md#list-of-all-modules) you use, you need to [import the required modules](@/guides/building-and-testing/modules.md#importing-required-modules).

In the entry point file of your application, import the `handsontable/base` module:
```js
// your `main.js` file
import { createApp } from 'vue';
import App from './App.vue';

// import the `handsontable/base` module
import Handsontable from 'handsontable/base';
```

### Step 2: Import optional modules

Import optional modules of your choice:

- [List of all modules &#8594;](@/guides/building-and-testing/modules.md#list-of-all-modules)
- [Importing and registering all modules &#8594;](@/guides/building-and-testing/modules.md#importing-and-registering-all-modules)

For example, to use the [`numeric`](@/guides/cell-types/numeric-cell-type.md) cell type and the [`UndoRedo`](@/api/undoRedo.md) plugin:
```js
import { createApp } from 'vue';
import App from './App.vue';

import Handsontable from 'handsontable/base';

import {
  // import the `NumericCellType` module
  NumericCellType,
} from 'handsontable/cellTypes';

import {
  // import the `UndoRedo` module
  UndoRedo,
} from 'handsontable/plugins';
```

### Step 3: Import your modules' registering methods

Import the registering methods that let you register your chosen modules:

- [Importing and registering all modules &#8594;](@/guides/building-and-testing/modules.md#importing-and-registering-all-modules)

For example, to import the registering methods of the [`numeric`](@/guides/cell-types/numeric-cell-type.md) cell type and of the [`UndoRedo`](@/api/undoRedo.md) plugin:
  ```js
  import Vue from 'vue';
  import App from './App.vue';

  import Handsontable from 'handsontable/base';

  import {
    NumericCellType,
    // import the `registerCellType()` method
    registerCellType,
  } from 'handsontable/cellTypes';

  import {
    UndoRedo,
    // import the `registerPlugin()` method
    registerPlugin,
  } from 'handsontable/plugins';
  ```

### Step 4: Register your modules
Register your chosen modules, using the registering methods.

For example, to register the [`numeric`](@/guides/cell-types/numeric-cell-type.md) cell type and the [`UndoRedo`](@/api/undoRedo.md) plugin: 
```js
import { createApp } from 'vue';
import App from './App.vue';
import router from './router';

import Handsontable from 'handsontable/base';

import {
  registerCellType,
  NumericCellType,
} from 'handsontable/cellTypes';

import {
  registerPlugin,
  UndoRedo,
} from 'handsontable/plugins';

// register the `NumericCellType` module
registerCellType(NumericCellType);

// register the `UndoRedo` module
registerPlugin(UndoRedo);

createApp(App).use(router).mount('#app');
```