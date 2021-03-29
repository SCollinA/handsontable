---
title: Using the hot-column component
permalink: /8.5/vue-hot-column
canonicalUrl: /vue-hot-column
---

# {{ $frontmatter.title }}

[[toc]]

::: tip
Version `4.1.0` of the `@handsontable/vue` wrapper introduces a new feature - a `hot-column` component.
:::

It doesn't only allow to configure the column-related settings using the `hot-column` component's attributes, but also create custom renderers and editors using Vue components.

## Declaring column settings

To declare column-specific settings, simply pass the settings as `hot-column` props (either separately or wrapped as a `settings` prop, exactly as you would for `hot-table`).

```html
<hot-table :data="hotData">
  <hot-column title="First column header">
  </hot-column>
  <hot-column :settings="secondColumnSettings" read-only="true">
  </hot-column>
</hot-table>
```

```js
import Vue from 'vue';
import { HotTable, HotColumn } from '@handsontable/vue';
import Handsontable from 'handsontable';

new Vue({
  el: '#example1',
  data: function() {
    return {
      hotData: Handsontable.helper.createSpreadsheetData(10, 10),
      secondColumnSettings: {
        title: 'Second column header'
      }
    }
  },
  components: {
    HotTable,
    HotColumn
  }
});
```

## Object data source

To work with an array of objects for the `hot-column` component you need to provide precise information about the data structure for columns. To do so, refer to the data for a column in props as `data`.

```html
<hot-table :data="hotData">
  <hot-column title="ID" data="id">
  </hot-column>
  <hot-column :settings="secondColumnSettings" read-only="true" data="name">
  </hot-column>
  <hot-column title="Price" data="payment.price">
  </hot-column>
  <hot-column title="Currency" data="payment.currency">
  </hot-column>
</hot-table>
```
```js
import Vue from 'vue';
import { HotTable, HotColumn } from '@handsontable/vue';
import Handsontable from 'handsontable';

new Vue({
  el: '#example2',
  data: function() {
    return {
      hotData: [
        {id: 1, name: 'Table tennis racket', payment: {price: 13, currency: 'PLN'}},
        {id: 2, name: 'Outdoor game ball', payment: {price: 14, currency: 'USD'}},
        {id: 3, name: 'Mountain bike', payment: {price: 300, currency: 'USD'}}
      ],
      secondColumnSettings: {
        title: 'Second column header'
      }
    }
  },
  components: {
    HotTable,
    HotColumn
  }
});
```

## Declaring a custom renderer as a component

The wrapper allows creating custom renderers using Vue components. The data you would normally get as arguments of the rendering function will be injected into the rendering component's `$data` object.

To mark a component as a Handsontable renderer, simply add a `hot-renderer` attribute to it.

::: warning
Because the Handsontable's `autoRowSize` and `autoColumnSize` options require calculating the widths/heights of some of the cells before rendering them into the table, it's not currently possible to use them alongside component-based renderers, as they're created after the table's initialization.
**Be sure to turn those options off in your Handsontable config, as keeping them enabled may cause unexpected results.**
:::

```html
<hot-table :settings="hotSettings">
  <hot-column :width="250">
    <custom-renderer hot-renderer></custom-renderer>
  </hot-column>
</hot-table>
```
```js
import Vue from 'vue';
import { HotTable, HotColumn } from '@handsontable/vue';
import Handsontable from 'handsontable';

const CustomRenderer = {
  template: '<div><i style="color: #a9a9a9">Row: {{row}}, column: {{col}},</i> value: {{value}}</div>',
  data: function() {
    return {
      // We'll need to define properties in our data object,
      // corresponding to all of the data being injected from
      // the BaseEditorComponent class, which are:
      // - hotInstance (instance of Handsontable)
      // - row (row index)
      // - col (column index)
      // - prop (column property name)
      // - TD (the HTML cell element)
      // - cellProperties (the cellProperties object for the edited cell)
      hotInstance: null,
      TD: null,
      row: null,
      col: null,
      prop: null,
      value: null,
      cellProperties: null
    }
  }
};

const App = new Vue({
  el: '#custom-renderer-example',
  data: function() {
    return {
      hotSettings: {
        data: Handsontable.helper.createSpreadsheetData(10, 10)  ,
        licenseKey: 'non-commercial-and-evaluation',
        autoRowSize: false,
        autoColumnSize: false
      }
    }
  },
  components: {
    HotTable,
    HotColumn,
    CustomRenderer
  }
});
```

**Note:** in order for the cell renderers to be independent, there are renderer components created for each of the displayed cells (all of them being clones of the "original" renderer component). For performance reasons, there are cached using the LRU algorithm, which stores a certain amount of entries, and overwrites the least recently used ones with fresh ones.
By default, the number of entries available for the cache is set to `3000`, which means 3000 cells can be rendered at the same time, while being read from the cache. However, for larger tables, some of the cells may not be able to be cached, and thus, their corresponding component would be recreated each time a cell is rendered (which is not great for performance).

To prevent this problem, it is possible to pass the `**wrapperRendererCacheSize**` option to the `HotTable` component and set it to a number of entries to be available in the renderer cache.

## Declaring a custom editor as a component

You can also utilize the Vue components to create custom editors. To do so, you'll need to create a component compatible with Handsontable's editor class structure. The easiest way to do so is to extend `BaseEditorComponent` - a base editor component exported from `@handsontable/vue`.

This will give you a solid base to build upon. Note, that the editor component needs to tick all of the boxes that a regular editor does, such as defining the `getValue`, `setValue`, `open`, `close` and `focus` methods, which are abstract in the `BaseEditor`. For more info, check the documentation on [creating custom editors from scratch](cell-editor#selecteditor---creating-editor-from-scratch).

```html
<hot-table :settings="hotSettings">
  <hot-column :width="250">
    <custom-editor hot-editor></custom-editor>
  </hot-column>
</hot-table>

<script type="text/x-template" id="editor-template">
  // We're binding the `style` attribute to the style object in our component's data
  // as well as the `mousedown` event to a function, which stops the event propagation
  // in order to prevent closing the editor on click.
  <div v-if="isVisible" id="editorElement" :style="style" @mousedown="stopMousedownPropagation" >
    <button v-on:click="setLowerCase">{{ value.toLowerCase() }}</button>
    <button v-on:click="setUpperCase">{{ value.toUpperCase() }}</button>
  </div>
</script>
```
```js
import Vue from 'vue';
import { HotTable, HotColumn, BaseEditorComponent } from '@handsontable/vue';
import Handsontable from 'handsontable';

const CustomEditor = {
  name: 'CustomEditor',
  template: '#editor-template',
  extends: BaseEditorComponent,
  data: function() {
    return {
      // We'll need to define properties in our data object,
      // corresponding to all of the data being injected from
      // the BaseEditorComponent class, which are:
      // - hotInstance (instance of Handsontable)
      // - row (row index)
      // - col (column index)
      // - prop (column property name)
      // - TD (the HTML cell element)
      // - originalValue (cell value passed to the editor)
      // - cellProperties (the cellProperties object for the edited cell)
      hotInstance: null,
      TD: null,
      row: null,
      col: null,
      prop: null,
      originalValue: null,
      value: '',
      cellProperties: null,
      isVisible: false,
      style: {
        position: 'absolute',
        padding: '15px',
        background: '#fff',
        zIndex: 999,
        border: '1px solid #000'
      }
    }
  },
  methods: {
    stopMousedownPropagation: function(e) {
      e.stopPropagation();
    },
    prepare: function(row, col, prop, td, originalValue, cellProperties) {
      // We'll need to call the `prepare` method from
      // the `BaseEditorComponent` class, as it provides
      // the component with the information needed to use the editor
      // (hotInstance, row, col, prop, TD, originalValue, cellProperties)
      BaseEditorComponent.options.methods.prepare.call(this, row, col, prop, td, originalValue, cellProperties);

      if (!document.body.contains(this.$el)) {
        document.body.appendChild(this.$el);
      }

      const tdPosition = td.getBoundingClientRect();

      // As the `prepare` method is triggered after selecting
      // any cell, we're updating the styles for the editor element,
      // so it shows up in the correct position.
      this.style.left = tdPosition.left + window.pageXOffset + 'px';
      this.style.top = tdPosition.top + window.pageYOffset + 'px';
    },
    setLowerCase: function() {
      this.setValue(this.value.toLowerCase());
      this.finishEditing();
    },
    setUpperCase: function() {
      this.setValue(this.value.toUpperCase());
      this.finishEditing();
    },
    open: function() {
      this.isVisible = true;
    },
    close: function() {
      this.isVisible = false;
    },
    setValue: function(value) {
      this.value = value;
    },
    getValue: function() {
      return this.value;
    }
  }
};

const App = new Vue({
  el: '#custom-editor-example',
  data: function() {
    return {
      hotSettings: {
        data: [
          ['Obrien Fischer'], ['Alexandria Gordon'], ['John Stafford'], ['Regina Waters'], ['Kay Bentley'], ['Emerson Drake'], ['Deann Stapleton']
        ],
        licenseKey: 'non-commercial-and-evaluation',
        rowHeaders: true
      }
    }
  },
  components: {
    HotTable,
    HotColumn,
    CustomEditor
  }
});
```

## Using the renderer/editor components with `v-model`

You can obviously use Vue's `v-model` with the renderer and editor components.

In the example below, we're utilizing an input with `v-model` assigned, and the reading the bound property from the renderer component to highlight the rows entered into the input.

List of row indexes (starting from 0):

```html
<div id="v-model-example">
  <label for="mainInput">List of row indexes (starting from 0):</label><br>
    <input id="mainInput" v-model="highlightedRows"/>

    <br><br>

    <hot-table :settings="hotSettings" :row-headers="true" :col-headers="true">
      <hot-column :width="50">
        <custom-renderer hot-renderer></custom-renderer>
      </hot-column>
    </hot-table>
</div>
```
```js
import Vue from 'vue';
import { HotTable, HotColumn } from '@handsontable/vue';
import Handsontable from 'handsontable';

const CustomRenderer = {
  template: `<div v-bind:style="{ backgroundColor: bgColor }">{{value}}</div>`,
  data: function() {
    return {
      hotInstance: null,
      TD: null,
      row: null,
      col: null,
      prop: null,
      value: null,
      cellProperties: null
    }
  },
  computed: {
    bgColor: function() {
      console.log(this.$root.highlightedRows);
      return this.$root.highlightedRows.includes(this.row) ? '#40b882' : '#fff';
    }
  }
};

const App = new Vue({
  el: '#v-model-example',
  data: function() {
    return {
      hotSettings: {
        data: Handsontable.helper.createSpreadsheetData(10, 1)  ,
        licenseKey: 'non-commercial-and-evaluation',
        autoRowSize: false,
        autoColumnSize: false
      },
      highlightedRows: ''
    }
  },
  components: {
    HotTable,
    HotColumn,
    CustomRenderer
  }
});
```

## A more advanced example

In this example, we'll be combining several capabilities of the wrapper:

1. We will create a custom editor component with an external dependency, which will act as both renderer and editor
2. We will declare settings for several columns using Vue's `v-for`
3. We will create a component, which state will be bound by the data we get from the first component

Due to the complexity of this example, I've decided to split the components to different files, so it's previewable on Codesandbox instead of jsfiddle.

<iframe src="https://codesandbox.io/embed/advanced-vue-hot-column-implementation-d4ymm?fontsize=14" title="Advanced vue hot-column implementation (7.2.2 + 4.1.1)" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb" style={{
  width:'100%',
  height: 500,
  border: 0,
  borderRadius: 4,
  overflow: 'hidden',
}} sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

### 1. Editor component with an external dependency, which will act as both renderer and editor

To use an external editor component with Handsontable, you'll need to create an addition "bridge" component, to connect your dependency's and Handsontable's API. In this example, we'll use an external color-picker component, [vue-color](https://github.com/xiaokaike/vue-color).

The editor implementation is pretty straightforward: you need to import your dependency, place in in your editor template and attach its events to your editor logic and you're done!
In our case, we're also adding an "Apply" button, which triggers Handsontable base editor's `finishEditing` method, so all the heavy lifting regarding passing the new value to the dataset is done for us.

All that's left is to modify the component template to be used as a renderer _and_ editor. We'll utilize the `isEditor` and `isRenderer` properties, injected to the component instances created by the wrapper. The template will be divided into a render part and an editor part using Vue's `v-if`.

This component contains some Vuex state logic, but ignore it for now, we'll get to it in the third point.

<iframe src="https://codesandbox.io/embed/advanced-vue-hot-column-implementation-d4ymm?fontsize=14&hidenavigation=1&module=%2Fsrc%2FColorPicker.vue&view=editor" title="Advanced @handsontable/vue hot-column implementation" style={{
  width:'100%',
  height: 500,
  border: 0,
  borderRadius: 4,
  overflow: 'hidden',
}} sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

### 2. Using `v-for` for column declaration

Let's use `v-for` to declare the second and third column in a loop. Obviously, you can bind the loop to your data and get the settings from there.

```html
<hot-table :settings="hotSettings">
  <hot-column :width="120">
    <stars-rating hot-renderer></stars-rating>
  </hot-column>
  <hot-column v-for="n in 2" :width="120" v-bind:key="'col' + n">
    <color-picker hot-editor hot-renderer></color-picker>
  </hot-column>
</hot-table>
```

### 3. Binding the state between components.

As you can see in our first editor/renderer component, we're already commiting all of the changes into the applications `$store`. This way, we can easily bind the state of our new component (based on a star-rating component dependency) to the data in the second and third column.

<iframe src="https://codesandbox.io/embed/advanced-vue-hot-column-implementation-d4ymm?fontsize=14&hidenavigation=1&module=%2Fsrc%2FStarsRating.vue&view=editor" title="Advanced @handsontable/vue hot-column implementation" style={{
  width:'100%',
  height: 500,
  border: 0,
  borderRadius: 4,
  overflow: 'hidden',
}} sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>