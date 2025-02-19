# Vue Tables 2

[![npm version](https://badge.fury.io/js/vue-tables-2.svg)](https://badge.fury.io/js/vue-tables-2) [![GitHub stars](https://img.shields.io/github/stars/matfish2/vue-tables-2.svg)](https://github.com/matfish2/vue-tables-2/stargazers) [![GitHub license](https://img.shields.io/badge/license-GPLv3-blue.svg)](https://raw.githubusercontent.com/matfish2/vue-tables-2/master/LICENSE) [![npm](https://img.shields.io/npm/dt/vue-tables-2.svg)](https://www.npmjs.com/package/vue-tables-2) [![Build Status](https://travis-ci.org/matfish2/vue-tables-2.svg?branch=master)](https://travis-ci.org/matfish2/vue-tables-2) [![](https://data.jsdelivr.com/v1/package/npm/vue-tables-2/badge)](https://www.jsdelivr.com/package/npm/vue-tables-2) [![Gitter chat](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/vue-tables-2/Lobby)

Important Note: As of version 1.6.0 the license for the package has changed from MIT to GPLv3. 
Users of previous versions can of course keep using the package under MIT license.
If you require an MIT license for commercial use you can purchase it [here](https://cp.xscode.com/repositories/13).
In addition to the permissive license, subscribed users will receive ongoing priority support for any issue that might arise.

[Click here](https://jsfiddle.net/matfish2/jfa5t4sm/) to see the client component in action and fiddle with the various [options](#options)
or [here](https://jsfiddle.net/matfish2/js4bmdbL/) for a rudimentary server component demo


**Recently Added: Resizable Columns, Editable Cells, Hidden Columns, Improved Accessibility, and more**


- [Usage](#usage)
- [Dependencies](#dependencies)
- [Installation](#installation)
- [Client Side](#client-side)
  - [Grouping](#grouping)
- [Server Side](#server-side)
  - [Implementations](#implementations)
- [Nested Data](#nested-data)
- [Templates](#templates)
  - [Scoped Slots](#scoped-slots)
  - [Virtual DOM Functions](#virtual-dom-functions)
  - [Vue Components](#vue-components)
- [Editable Cells](#editable-cells)
- [Child Rows](#child-rows)
- [Methods](#methods)
- [Properties](#properties)
- [Events](#events)
- [Date Columns](#date-columns)
- [Filters Algorithm](#filters-algorithm)
- [Custom Filters](#custom-filters)
- [Client Side Filters](#client-side-filters)
- [Server Side Filters](#server-side-filters)
- [List Filters](#list-filters)
- [Columns Visibility](#columns-visibility)
- [Cells Conditional Styling](#cells-conditional-styling)
- [Custom Sorting](#custom-sorting)
- [Client Side Sorting](#client-side-sorting)
- [Server Side Sorting](#server-side-sorting)
- [Multiple Sorting](#multiple-sorting)
- [Slots](#slots)
- [Options](#options)
- [VueJS 1](#vuejs-1)

# Usage

## Dependencies

* Vue.js (>=2.0)
* Server Side: axios OR vue-resource (>=0.9.0) OR jQuery for the AJAX requests

## Compatibility

* Vuex (>=2.0)
* Bootstrap 3 / Bootstrap 4 / Bulma

## Installation

```bash
npm install vue-tables-2
```

Require the script:

```js
import {ServerTable, ClientTable, Event} from 'vue-tables-2';
```

### Register the component(s)

```js
Vue.use(ClientTable, [options = {}], [useVuex = false], [theme = 'bootstrap3'], [template = 'default']);
```

Or/And:

```js
Vue.use(ServerTable, [options = {}], [useVuex = false], [theme = 'bootstrap3'], [template = 'default']);
```

* `useVuex` is a boolean indicating whether to use `vuex` for state management, or manage state on the component itself.
If you set it to `true` you must add a `name` prop to your table, which will be used to register a module on your store.
Use `vue-devtools` to look under the hood and see the current state.

> Note: If you are using `vue-router` or simply toggling the table with `v-if`, set the `preserveState` option to `true`.

* `theme` Use this option to select a CSS framework. Options:'bootstrap3','bootstrap4','bulma'. 
You can also pass you own theme. Use a file from the `themes` folder as boilerplate.

* `template` Use this option to select an HTML template. Currently supported: 'default', 'footerPagination'
You can also pass your own template. Use a file from the `templates` folder as boilerplate.

> Note: You may need to add a little styling of your own. 
If you come up with some improvments to the templates or themes, which brings them closer to the optimum, you are welcome to send a PR.

> Note: The template is written using `jsx`, so you will need a [jsx compiler](https://github.com/vuejs/babel-plugin-transform-vue-jsx) to modify it (the package is using the compiled version under the `compiled` folder).

### Using Script Tag

If you are not using NPM you can also import the minified version found in `dist/vue-tables-2.min.js`.
Copy the file into your project and import it:

```html
<script src="/path/to/vue-tables-2.min.js"></script>
```

Or, if you prefer, use the [CDN version](https://www.jsdelivr.com/package/npm/vue-tables-2?path=dist).

This will expose a global `VueTables` object containing `ClientTable`, `ServerTable` and `Event` as properties.

E.g:

```js
Vue.use(VueTables.ClientTable);
```

## Client Side

Add the following element to your page wherever you want it to render.
Make sure to wrap it with a parent element you can latch your vue instance into.

```html
<div id="people">
  <v-client-table :data="tableData" :columns="columns" :options="options"></v-client-table>
</div>
```

Create a new Vue instance (You can also nest it within other components). An example works best to illustrate the syntax:

```js
new Vue({
    el: "#people",
    data: {
        columns: ['id', 'name', 'age'],
        tableData: [
            { id: 1, name: "John", age: "20" },
            { id: 2, name: "Jane", age: "24" },
            { id: 3, name: "Susan", age: "16" },
            { id: 4, name: "Chris", age: "55" },
            { id: 5, name: "Dan", age: "40" }
        ],
        options: {
            // see the options API
        }
    }
});
```

You can access the filtered dataset at any given moment by fetching the `filteredData` computed property of the table, using `ref` as a pointer (`this.$refs.myTable.filteredData`); This will return the current page. To access the entire filtered dataset use `allFilteredData` instead.

> Important: when loading data asynchronously add a `v-if` conditional to the component along with some `loaded` flag, so it will only compile once the data is attached.

### Grouping

The client component supports simple grouping of rows by a common value. By simple we mean that the grouping is merely *presentational*, and not backed by a real model-level data grouping (i.e the data is NOT divided into distinct arrays). you can group by any property on your dataset. For example, for a table of countries you can group by a `continent` property. Simply declare in your options `groupBy:'continent'`.

If you want to present some additional meta-data realted to each value, you can use the `groupMeta` option, along with the dedicated `__group_meta` scoped slot. For example:

```js
groupBy:'continent',
groupMeta:[
   {
       value:'Africa',
       data:{
           population:1216,
           countries:54
       }
   },
   {
       value:'Asia',
       data:{
           population:4436
           countries:48
       }
   },
   {
       value:'Europe',
       data:{
           population:741.4,
           countries:50
       }
   }
   // etc...
]
```

```vue
<span slot="__group_meta" slot-scope="{ value, data }">
  {value} has {data.countries} countries and a population of {data.population} million
</span>
```
## Server side

```html
<div id="people">
  <v-server-table url="/people" :columns="columns" :options="options"></v-server-table>
</div>
```

Javascript:

```js
new Vue({
    el: "#people",
    data: {
        columns: ['id', 'name', 'age'],
        options: {
            // see the options API
        }
    }
});
```

All the data is passed in the following GET parameters:
* `query`,
* `limit`,
* `page`,
* `orderBy`,
* `ascending`,
* `byColumn`.

You need to return a JSON object with two properties:
* `data` : `array` - An array of row objects with identical keys.
* `count`: `number` - Total count before limit.

> Note: If you are calling a foreign API or simply want to use your own keys, refer to the `responseAdapter` option.

### Custom Request Function

by default the library supports `JQuery`, `vue-resource` and `axios` as ajax libraries.
If you wish to use a different library, or somehow alter the request (e.g add auth headers, or manipulate the data) use the `requestFunction` option. E.g:

```js
options: {
    requestFunction: function (data) {
        return axios.get(this.url, {
            params: data
        }).catch(function (e) {
            this.dispatch('error', e);
        }.bind(this));
    }
}
```

Note: when using a custom request function, the `url` prop is not required.

### Implementations

I have included [an Eloquent implementation](https://raw.githubusercontent.com/matfish2/vue-tables-2/master/server/PHP/EloquentVueTables.php) for Laravel Users.

If you happen to write other implementations for PHP or other languages, a pull request would be most welcome, under the following guidelines:

1. Include the class under `./server/{language}`.
1. Name it according to convention: `{concrete}VueTables`.
1. if this is the first implementation in this language add an interface similar to the one found in the PHP folder.
1. Have it implement the interface.
1. TEST IT.

# Nested Data

Oftentimes your dataset will consist of *objects*, rather than primitive types.
By default, the package has no knowledge of how those objects should be presented.
To tell the package how to handle those fields you can use one of two options, depending on your needs:

1. Decide which *primitive* property you would like to refer to as the relevant piece of data, by using the dot notation when declaring you `columns` prop. E.g: `['name','age','meta.education.degree']` 
2. Use [Templates](#templates)

Option 1 is more simple and straight-forward. However, it disregards all the other properties, which means that sorting, filtering and presentation will all refer to the single piece of primitive data at the "end of the chain".

If you want to use the entire object, option 2 is your best route. This will allow you to incorporate all the properties in the presentation. When using the client component note that:
* Default filtering behaviour will recursively scan the entire object for the query. 
* If the column is sortable, you will need to define a sorting algorithm, using the `customSorting`[#custom-sorting] option. 

# Templates

Templates allow you to wrap your cells with vue-compiled HTML. It can be used in any of the following ways:

## Scoped Slots
If you are using Vue 2.1.0 and above, you can use [scoped slots](https://vuejs.org/v2/guide/components.html#Scoped-Slots) to create templates:

```vue
<v-client-table :data="entries" :columns="['id', 'name' ,'age', 'edit']">
      <a slot="edit" slot-scope="props" class="fa fa-edit" :href="edit(props.row.id)"></a>
</v-client-table>
```

Note: You can get the index of the current row relative to the entire data set using `props.index`

## Virtual DOM Functions

The syntax for Virtual DOM function is similar to that of `render` functions, as it leverages the virtual DOM to bind the templates into the main table template.

If you choose this option, it is recommended to use JSX, which closely resembles HTML, to write the templates (To compile `jsx` you need to install the [vue jsx transform](https://github.com/vuejs/babel-plugin-transform-vue-jsx)).

E.g.:

```js
data: {
    columns: ['erase'],
    options: {
        ...
        templates: {
            erase: function (h, row, index) {
                return <delete id = {row.id}>< /delete>
            }
        }
        ...
    }
}
```

The first parameter is the `h` scope used to compile the element. It MUST be called `h`.
The second parameter gives you access to the row data.
In addition a `this` context will be available, which refers to the root vue instance. This allows you to call your own instance methods directly.
Note: when using a `.vue` file `jsx` must be imported from a dedicated `.jsx` file in order to compile correctly. E.g

edit.jsx

```js
export default function(h, row, index) {
return <a class='fa fa-edit' href={'#/' + row.id + '/edit'}></a>
}
```

app.vue

```html
<script>
  import edit from './edit'

  templates: {
      edit
  }
</script>
```

## Vue Components
Another option for creating templates is to encapsulate the template within a component and pass the name. The component must have a `data` property, which will receive the row object. You can also add an optional `index` prop, to get the non-zero-based index of the current row relative to the entire dataset, and an optional `column` prop to get the current column. E.g:

```js
Vue.component('delete', {
    props: ['data', 'index', 'column'],
    template: `<a class='delete' @click='erase'></a>`,
    methods: {
        erase() {
            let id = this.data.id; // delete the item
        }
    }
});
```

```js
options: {
    ...
    templates: {
        erase: 'delete'
    }
    ...
}
```

This method allows you to also use single page .vue files for displaying the template data
E.g:
edit.vue

```html
<template>
  <a class="fa fa-edit" :href="edit(data.id)">Edit</a>
</template>

<script>
  export default {
      props: ['data', 'index'],
  }
</script>
```

app.vue

```html
<script>
  import edit from './edit'

  templates: {
      edit
  }
</script>
```

**Important**:
* Templates must be declared in the `columns` prop
* Don't include HTML directly in your dataset, as it will be parsed as plain text.

# Editable Cells 

Editable cells are currently supported only for the client table.
To ensure editable data is reflected on your original dataset you must use `v-model` instead of the `data` prop.
Each row in dataset must have a unique identifier. By default its key is set to `id`. Use the `uniqueKey` option if your dataset has a different identifier.

As always examples work best to illustrate the syntax:

Start by declaring the column(s) you wish to be editable using the `editableColumns` option:

```js
{
   editableColumns:['text','text2','checkbox']
}
````

Then use scoped slots to build the logic:

```vue
 <v-client-table :columns="client.columns" :options="client.options" v-model="client.data">
    // update text on the fly
    <input type="text" slot="text" slot-scope="{row, update}" v-model="row.text" @input="update">
    // update a checkbox
    <input type="checkbox" slot="checkbox" slot-scope="{row, update}" v-model="row.checkbox" @input="update">
    
    // update text on submit + toggle editable state + revert to original value on cancel
    <div slot="text2" slot-scope="{row, update, setEditing, isEditing, revertValue}">
     <span @click="setEditing(true)" v-if="!isEditing()">
         {{row.text2}}
     </span>
        <span v-else>
     <input type="text" v-model="row.text2">
        <button type="button" @click="update(row.text2); setEditing(false)">Submit</button>
   <button type="button" @click="revertValue(); setEditing(false)">Cancel</button>        
</span>
    </div>           
</v-client-table>
```

In addition to the `input` event which is responsible - in conjunction with `v-model` - for updating the dataset, an additional `update` event is triggered with the following payload:
```{row, column, oldVal, newVal}```

# Child Rows

Child rows allow for a custom designed output area, namely a hidden child row underneath each row, whose content you are free to set yourself.

When using the `childRow` option you must pass a unqiue `id` property for each row, which is used to track the current state.
If your identifier key is not `id`, use the `uniqueKey` option to set it.

The syntax is identical to that of templates:

Using Scoped Slots:

```vue
<template slot="child_row" scope="props">
  <div><b>First name:</b> {{props.row.first_name}}</div>
  <div><b>Last name:</b> {{props.row.last_name}}</div>
</template>
```

Using a function and (optional) JSX:

```js
    options:{
    ...
    childRow: function(h, row) {
      return <div>My custom content for row {row.id}</div>
    }
    ...
}
```

Using a component name or a `.vue` file: (See [Templates](#templates) above for a complete example)

```js
    options:{
    ...
        childRow: 'row-component'
    ...
}
```

When the plugin detects a `childRow` function it appends the child rows and prepends to each row an additional toggler column with a `span` you can design to your liking.

Example styling (also found in `style.css`):
```css
.VueTables__child-row-toggler {
    width: 16px;
    height: 16px;
    line-height: 16px;
    display: block;
    margin: auto;
    text-align: center;
}

.VueTables__child-row-toggler--closed::before {
    content: "+";
}

.VueTables__child-row-toggler--open::before {
    content: "-";
}
```

You can also trigger the child row toggler programmtically. E.g, to toggle the row with an id of 4:

```js
this.$refs.myTable.toggleChildRow(4); // replace myTable with your own ref
```

Alternatively, you can use the `showChildRowToggler` option to prevent the child row toggler cells from being rendered. (See [options](#options))

# Methods

Call methods on your instance using the [`ref`](http://vuejs.org/api/#ref) attribute.

* `setPage(page)`
* `setLimit(recordsPerPage)`
* `setOrder(column, isAscending)`
* `setFilter(query)` - `query` should be a string, or an object if `filterByColumn` is set to `true`.
* `resetQuery()` - Resets all query inputs (user-request filters) to empty strings.
* `getData()` Get table data using the existing request parameters. Server component only.
* `refresh()` Refresh the table. This method is simply a wrapper for the `serverSearch` method, and thus resets the pagination. Server component only
* `getOpenChildRows(rows = null)` 
If no argument is supplied returns all open child row components in the page. 
To limit the returned dataset you can pass the `rows` arguemnt, which should be an array of unique identifiers.

Note: 
A. This method is only to be used when the child row is a component. 
B. In order for this method to work you need to set the `name` property on your component to `ChildRow`

# Properties

Get properties off your instance using the [`ref`](http://vuejs.org/api/#ref) attribute.

* `openChildRows` `array` The ids of all the currently open child rows

# Events

Using Custom Events (For child-parent communication):

```html
<v-server-table :columns="columns" url="/getData" @loaded="onLoaded"></v-server-table>
```

* Using the event bus:

```js
Event.$on('vue-tables.loaded', function (data) {
    // Do something
});
```

Note: If you are using the bus and want the event to be "namespaced", so you can distinguish bewteen different tables on the same page, use the `name` prop.
The event name will then take the shape of `vue-tables.tableName.eventName`.

* Using Vuex:

```js
mutations: {
    ['tableName/LOADED'](state, data) {
        // Do something
    }
}
```

* `vue-tables.filter` / `tableName/FILTER`

Fires off when a filter is changed. Sends through the name and value in case of column filter, or just the value in case of the general filter

* `vue-tables.filter::colName`

Same as above, only this one has the name attached to the event itself, and only sends through the value.
Releveant only for non-vuex tables with `filterByColumn` set to true.

* `vue-tables.sorted / `tableName/SORTED`

Fires off when the user sorts the table. Sends through the column and direction.
In case of multisorting (Shift+Click) an array will be sent sorted by precedence.

* `vue-tables.loading` / `tableName/LOADING` (server)

Fires off when a request is sent to the server. Sends through the request data.

* `vue-tables.loaded` / `tableName/LOADED` (server)

Fires off after the response data has been attached to the table. Sends through the response.

You can listen to those complementary events on a parent component and use them to add and remove a *loading indicator*, respectively.

* `vue-tables.pagination` / `tableName/PAGINATION`

Fires off whenever the user changes a page. Send through the page number.

* `vue-tables.limit` / `tableName/LIMIT`

Fires off when the per page limit is changed

* `vue-tables.error` / `tableName/ERROR` (server)

Fires off if the server returns an invalid code. Sends through the error

* `vue-tables.row-click` / `tableName/ROW_CLICK`

Fires off after a row was clicked. sends through the row and the mouse event.
When using the client component, if you want to recieve the *original* row, so that it can be directly mutated, you must have a unique row identifier.
The key defaults to `id`, but can be changed using the `uniqueKey` option.

# Date Columns

For date columns you can use [daterangepicker](https://github.com/dangrossman/bootstrap-daterangepicker) as a filter instead of the normal free-text input. Note that you must import all the dependencies (JQuery + moment.js + daterangepicker) GLOBALLY, i.e on the `window` scope. You can check if this is so by typing `typeof $().daterangepicker` in the console. If it returns `function` you are good to go.
To tell the plugin which columns should be treated as date columns use the `dateColumns` option. (E.g: `dateColumns:['created_at','executed_at']`).

For the client component the date must be rendered as a `moment` object for the filtering to work. You can use the `toMomentFormat` option to do that for you. It accepts the current format of the dates in your dataset, and uses it to convert those dates to `moment` objects. E.g, if you have a `created_at` column with a date like `2017-12-31`, you can use `toMomentFormat: 'YYYY-MM-DD'`. To decide the presentation of the dates use the `dateFormat` option.

On the server component the filter will be sent along with the request in the following format: 

```js
{
    query:{
        created_at:{
            start: '2010-12-31 00:00:00',
            end: '2011-12-31 00:00:00'       
        }
    }
}
``` 

date presentation on the server component is completely up to you. If you are unable to control the server response, you can use the `templates` option to "massage" the date you get from the server into the desired format.

# Filters Algorithm

You can modify the default filtering algorithm per column using the `filterAlgorithm` option. 
For "fake" template columns which are not backed up by a real corresponding property this is a necessity, if you wish the column to be included in the search (either in generic mode or by column).

E.g, Say you have template column called `full_name` which combines first and last names; you can define the search algorithm like so:

```js
filterAlgorithm: {
  full_name(row, query) {
    return (row.first_name + ' ' + row.last_name).includes(query);
  }
}
```

# Custom Filters

Custom filters allow you to integrate your own filters into the plugin using Vue's events system.

## Client Side Filters

1. use the `customFilters` option to declare your filters, following this syntax:

```js
customFilters: [{
    name: 'alphabet',
    callback: function (row, query) {
        return row.name[0] == query;
    }
}]
```

1. Using the event bus:

```js
Event.$emit('vue-tables.filter::alphabet', query);
```

1. Using `vuex`:

```js
this.$store.commit('myTable/SET_CUSTOM_FILTER', {filter:'alphabet', value:query})
```

## Server Side Filters

A. use the `customFilters` option to declare your filters, following this syntax:

```js
customFilters: ['alphabet','age-range']
```

B. the same as in the client component.

# List Filters

When filtering by column (option `filterByColumn:true`), the `listColumns` option allows for filtering columns whose values are part of a list, using a select box instead of the default free-text filter.

For example:

```js
options: {
    filterByColumn: true,
    listColumns: {
        animal: [{
                id: 1,
                text: 'Dog'
            },
            {
                id: 2,
                text: 'Cat',
                hide:true
            },
            {
                id: 3,
                text: 'Tiger'
            },
            {
                id: 4,
                text: 'Bear'
            }
        ]
    }
}
```

> Note: The values of this column should correspond to the `id`'s passed to the list.
They will be automatically converted to their textual representation.

> Adding `hide:true` to an item, will exclude it from the options presented to the user

# Columns Visibility

If you would like to enable the user to control the columns' visibility set the `columnsDropdown` option to `true`.
This will add a dropdown button to the left of the per-page control. The drop down will contain a list of the columns with checkboxes to toggle visibility.

The `columnsDropdown` option can work in conjunction with `columnsDisplay`. The rule is that as long as the user hasn't toggled a column himself, the rules you have declared in `columnsDisplay` takes precedence. Once the user toggled a column, he is in charge of columns' visibility, and the settings of `columnsDisplay` are disregarded. 

# Cells Conditional Styling

The `cellClasses` option allows you to conditionally add class(es) to the `td` element based on predefined conditions.
For example, say you want to add a `low-balance` class to cells in a `balance` column that has a value of less than 100:

```js
cellClasses:{
  balance: [
    {
        class:'low-balance',
        condition: row => row.balance < 100
    }
  ]
}
```

# Custom Sorting

## Client Side Sorting

Sometimes you may one to override the default sorting logic which is applied uniformly to all columns.
To do so use the `customSorting` option. This is an object that recieves custom logic for specific columns.
E.g, to sort the `name` column by the last character:

```js
customSorting: {
    name: function (ascending) {
        return function (a, b) {
            var lastA = a.name[a.name.length - 1].toLowerCase();
            var lastB = b.name[b.name.length - 1].toLowerCase();

            if (ascending)
                return lastA >= lastB ? 1 : -1;

            return lastA <= lastB ? 1 : -1;
        }
    }
}
```

## Server Side Sorting

This depends entirely on your backend implemetation as the library sends the sorting direction trough the request.

# Multiple Sorting

Multiple sorting allows you to sort recursively by multiple columns.
Simply put, when the primary column (i.e the column the user is currently sorting) has two or more identical items, their order will be determined by a secondary column, and so on, until the list of columns is exhausted.

Example usage:
```js
{
    ...
    multiSorting: {
        name: [
            {
                column: 'age',
                matchDir: false
            },
            {
                column: 'birth_date',
                matchDir: true
            }
        ]
    }
    ...
}
```

The above code means that when the user is sorting by `name` and identical names are compared, their order will be determined by the `age` column. If the ages are also identical the `birth_date` column will determine the order.
The `matchDir` property tells the plugin whether the secondary sorting should match the direction of the primary column (i.e ascending/descending), or not.

In addition to programmatically setting the sorting in advance, by default the user can also use Shift+Click to build his own sorting logic in real time.
To disable this option set `clientMultiSorting` to `false`.

On the server component this behaviour is disabled by default, because it requires addtional server logic to handle the request.
To enable it set `serverMultiSorting` to `true`. The request will then contain a `multiSort` array, if applicable.

# Slots

Slots allow you to insert you own custom HTML in predefined positions within the component:

* `beforeTable`: Before the table wrapper. After the controls row
* `afterTable`: Before the table wrapper. 
* `beforeFilter`: Before the global filter (`filterByColumn: false`)
* `afterFilter`: After the global filter
* `afterFilterWrapper`: After the global filter wrapper
* `beforeSearch`: Before the search control
* `beforeLimit`: Before the per page control
* `afterLimit`: After the per page control
* `beforeFilters`: Before the filters row (`filterByColumn: true`)
* `afterFilters`: After the filters row
* `prependHead`: After the `<thead>` tag
* `beforeBody`: Before the `<tbody>` tag
* `afterBody`: After the `<tbody>` tag
* `prependBody`: Prepend to the `<tbody>` tag
* `appendBody`: Append to the `<tbody>` tag


In addition to these slots you can insert your own filter HTML in the filters row, or add content to the existing filter, e.g a button (When `filterByColumn` is set to `true`):

A. If you just want your own content make sure that the column is not filterable by omitting it from the `filterable` array.
Otherwise the slot content will be appended to the native filter.

B. Create a slot whose name is formatted `filter__{column}` (double underscore).

For example, to insert a checkbox on the `id` column instead of the normal input filter:

```js
{
  filterable:["name","age"] // omit the `id` column
}
```

```html
<div slot="filter__id">
    <input type="checkbox" class="form-control" v-model="allMarked" @change="markAll()">
</div>
 ```

# Options

Options are set in three layers, where the more particular overrides the more general.

1. Pre-defined component defaults.
2. Applicable user-defined defaults for the global Vue Instance. Passed as the second paramter to the `Use` statement.
3. Options for a single table, passed through the `options` prop.

Option | Type | Description | Default
-------|------|-------------|--------
caption | String | table `caption` element | `false`
childRow | Function| [See documentation](#child-rows) | `false`
childRowTogglerFirst | Boolean | Should the child row be positioned at the first column or the last one | `true`
clientMultiSorting | Boolean | Enable multiple columns sorting using Shift + Click on the client component | `true`
toggleGroups (client-side) | Boolean | When using the `groupBy` option, settings this to `true` will make group's visibility togglable, by turning the group name into a button | `false`
cellClasses | Object | See [documentation](#cells-conditional-styling) | `{}`
columnsClasses | Object | Add class(es) to the specified columns.<br> Takes key-value pairs, where the key is the column name and the value is a string of space-separated classes | `{}`
columnsDisplay | Object | Responsive display for the specified columns.<br><br> Columns will only be shown when the window width is within the defined limits. <br><br>Accepts key-value pairs of column name and device.<br><br> Possible values are `mobile` (x < 480), `mobileP` (x < 320), `mobileL` (320 <= x < 480), `tablet` (480 <= x < 1024), `tabletP` (480 <= x < 768), `tabletL` (768 <= x < 1024), `desktop` (x >= 1024).<br><br> All options can be preceded by the logical operators min,max, and not followed by an underscore.<br><br>For example, a column which is set to `not_mobile` will be shown when the width of the window is greater than or equal to 480px, while a column set to `max_tabletP` will only be shown when the width is under 768px | `{}`
columnsDropdown | Boolean | See [documentation](#columns-visibility) | `false`
customFilters | Array | See [documentation](#custom-filters) | `[]`
customSorting (client-side) | Object | See [documentation](#custom-sorting) | `{}`
dateColumns | Array | Use daterangepicker as a filter for the specified columns (when filterByColumn is set to true).<br><br>Dates should be passed as moment objects, or as strings in conjunction with the toMomentFormat option | `[]`
dateFormat | String | Format to display date objects (client component). Using [momentjs](https://momentjs.com/). This will also affect the datepicker on both components | `DD/MM/YYYY`
dateFormatPerColumn | Object | override the default date format for specific columns | `{}`
datepickerOptions | Object | Options for the daterangepicker when using a date filter (see dateColumns) | `{ locale: { cancelLabel: 'Clear' } }`
datepickerPerColumnOptions | Object | additional options for specific columns, to be merged with `datepickerOptions`. Expects an object where the key is a column name, and the value is an options object | `{}`
debounce | Number | Number of idle milliseconds (no key stroke) to wait before sending a request. Used to detect when the user finished his query in order to avoid redundant requests (server) or rendering (client) | `500`
descOrderColumns | Array | By default the initial sort direction is ascending. To override this for specific columns pass them into this option | `[]`
destroyEventBus | Boolean | Should the plugin destroy the event bus before the table component is destroyed? Since the bus is shared by all instances, only set this to `true` if you have a single instance in your page | `false`
editableColumns | Array | Columns that should be editable. See [documentation](#editable-cells) | `[]`
filterable | Array / Boolean | Filterable columns `true` - All columns. | Set to `false` or `[]` to hide the filter(s). Single filter mode (`filterByColumn:false`) is also affected
filterAlgorithm | Object | Define custom filtering logic per column. See [documentation](#filters-algorithm) | `{}`
footerHeadings | Boolean | Display headings at the bottom of the table too | `false`
headings | Object | Table headings. | Can be either a string or a function, if you wish to inject vue-compiled HTML.<br>E.g: `function(h) { return <h2>Title</h2>}`<br>Note that this example uses jsx, and not HTML.<br>The `this` context inside the function refers to the direct parent of the table instance.<br> Another option is to use a slot, named "h__{column}"<br>If no heading is provided the default rule is to extract it from the first row properties, where underscores become spaces and the first letter is capitalized
hiddenColumns | Array / Boolean | An array of columns to hide initially (can then be added by user using `columnsDropdown` option). Mutually exclusive with `visibleColumns` | `false`
groupBy (client-side) | String | Group rows by a common property. E.g, for a list of countries, group by the `continent` property | `false`
groupMeta (client-side) | Array | Meta data associated with each group value. To be used in conjunction with `groupBy`. See documentation for a full example | `[]`
headingsTooltips | Object | Table headings tooltips. | Can be either a string or a function, if you wish to inject vue-compiled HTML. Renders as `title` attribute of `<th>`. <br>E.g: `function(h) { return 'Expanded Title'}`<br>The `this` context inside the function refers to the direct parent of the table instance.
highlightMatches | Boolean | Highlight matches | `false`
initFilters | Object | Set initial values for all filter types: generic, by column or custom.<br><br> Accepts an object of key-value pairs, where the key is one of the following: <br><br>a. "GENERIC" - for the generic filter<br>b. column name - for by column filters.<br>c. filter name - for custom filters. <br><br>In case of date filters the date range should be passed as an object comprised of start and end properties, each being a moment object. | `{}`
initialPage | Number | Set the initial page to be displayed when the table loads | 1
listColumns | Object | See [documentation](#list-filters) | {}
multiSorting (client-side) | Object | See [documentation](#multiple-sotring) | {}
orderBy.ascending | Boolean | initial order direction | `orderBy: { ascending:true }`
orderBy.column | String | initial column to sort by | Original dataset order
pagination.chunk | Number | maximum pages in a chunk of pagination | `pagination: { chunk:10 }`
pagination.dropdown | Boolean | use a dropdown select pagination next to the records-per-page list, instead of links at the bottom of the table. | `pagination: { dropdown:false }`
pagination.nav | String | Which method to use when navigating outside of chunk boundries. Options are : `scroll` - the range of pages presented will incrementally change when navigating to a page outside the chunk (e.g 1-10 will become 2-11 once the user presses the next arrow to move to page 11). `fixed` - navigation will occur between fixed chunks (e.g 1-10, 11-20, 21-30 etc.). Double arrows will be added to allow navigation to the beginning of the previous or next chunk | `pagination: { nav: 'fixed' }` 
pagination.edge | Boolean | Show 'First' and 'Last' buttons | `pagination: { edge: false }`
params (server-side) | Object | Additional parameters to send along with the request | `{}`
perPage | number | Initial records per page | `10`
perPageValues | Array | Records per page options | `[10,25,50,100]`
preserveState | Boolean | Preserve dynamically created vuex module when the table is destroyed | `false`
requestAdapter (server-side) | Function | Set a custom request format | `function(data) { return data; }`
requestFunction (server-side) | Function | Set a custom request function | See documentation
requestKeys (server-side) | Object | Set your own request keys | `{ query:'query', limit:'limit', orderBy:'orderBy', ascending:'ascending', page:'page', byColumn:'byColumn' }`
resizableColumns | Boolean / Array | 3 options: a. `false` - no resizable columns b. `true` - all columns resizable c. array of resizable columns | `true`
responseAdapter (server-side) | Function | Transform the server response to match the format expected by the client. This is especially useful when calling a foreign API, where you cannot control the response on the server-side | `function(resp) { var data = this.getResponseData(resp); return { data: data.data, count: data.count } }`
rowAttributesCallback | Function | Add dynamic attributes to table rows.<br><br> E.g function(row) { return {id: row.id}} <br><br>This can be useful for manipulating the attributes of rows based on the data they contain | `{}`
rowClassCallback | Function | Add dynamic classes to table rows.<br><br> E.g function(row) { return `row-${row.id}`} <br><br>This can be useful for manipulating the appearance of rows based on the data they contain | `false`
saveState | Boolean | Constantly save table state and reload it each time the component mounts. When setting it to true, use the `name` prop to set an identifier for the table | `false`
sendEmptyFilters (server-side) | Boolean | When filtering by column send all request keys, including empty ones | `false` 
sendInitialRequest (server-side) | Boolean | Should a request to fetch the data be sent immediately when the table renders | `true`
serverMultiSorting | Boolean | Enable multiple columns sorting using Shift + Click on the server component | `false`
showChildRowToggler | Boolean | Enable render of `child row toggler` cell | `true`
skin | String | Space separated table styling classes | `table-striped table-bordered table-hover`
sortIcon | String | Sort icon classes | `{ base:'glyphicon', up:'glyphicon-chevron-up', down:'glyphicon-chevron-down', is:'glyphicon-sort' }`
sortable | Array |  Sortable columns | All columns
sortingAlgorithm | Function | define your own sorting algorithm  | `function (data, column) { return data.sort(this.getSortFn(column));}`
summary | String | Table's `summary` attribute for accessibility reasons | `false` 
storage | String | Which persistance mechanism should be used when saveState is set to true: `local` - localStorage. `session` - sessionStorage | `local`
templates | Object | See [documentation](#templates) | {}
texts | Object | see the `texts` object in [defaults.js](https://github.com/matfish2/vue-tables-2/blob/master/lib/config/defaults.js)</code>
toMomentFormat (client-side) | String | transform date columns string values to momentjs objects using this format. If this option is not used the consumer is expected to pass momentjs objects himself | `false`
uniqueKey | String | The key of a unique identifier in your dataset, used to track the child rows, and return the original row in row click event | `id`
visibleColumns | Array / Boolean | An array of columns to show initially (can then be altered by user using `columnsDropdown` option). Mutually exclusive with `hiddenColumns` | `false`

# VueJS 1

Users of VueJS 1 should use [this package](https://github.com/matfish2/vue-tables) .
