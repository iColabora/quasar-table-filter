# quasar-table-filter

## Install

```bash
npm i git+https://github.com/iColabora/quasar-table-filter.git#semver:^0.1.0
```

Create a file in boot with content:

```javascript
import QTableFilter from 'quasar-table-filter/src/components/QTableFilter'

export default ({ Vue }) => {
  Vue.component('QTableFilter', QTableFilter)
  Vue.component('q-table-filter', QTableFilter)
}
```

This compoent use ``` <q-table>``` base from [Quasar](https://quasar.dev/vue-components/table).

# Columns options

Like quasar default with follow properties:

|     Property     | Type    | Default | Values       | Description        |
|:-------------:|---------|---------|--------------|--------------------|
| filter_type   | String  | text    | text, select | Filter header type |
| disableFilter | Boolean | false   | true, false  | Hide filter column |


<span style="color:red">IMPORTANT: </span> Property ```name``` must be equal ```field``` property, except field of type ```function```, where property ```name``` must be equal first key in ```function```. Example:

```Javascript
[
    { name: 'protocol', required: true, label: 'Protocol', align: 'center', field: 'protocol' },
    { name: 'my_company', align: 'center', label: 'Company', field: row => row.my_company ? row.my_company.fantasyName : '', filter_type: 'select' }
]

```

# Component keys

|       Name      | Type    | Default    | Description                                                                                                                                                             |
|:---------------:|---------|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| data            | Array   | null       | Rows of data to display                                                                                                                                                 |
| columns         | Array   | null       | The column definitions (Array of Objects)                                                                                                                               |
| visible-columns | Array   | null       | Array of Strings defining column names ('name' property of each column from 'columns' prop definitions); Columns marked as 'required' are not affected by this property |
| classes         | String  |            | Classes in table                                                                                                                                                        |
| separator       | String  | horizontal | Use a separator/border between rows, columns or all cells                                                                                                               |
| header-filter   | Boolean | false      | Show after column name, a button to show a filter column                                                                                                                |
| columns-filter  | Boolean | false      | Show on header a line with filter each column                                                                                                                           |
| dense           | Boolean | false      | Dense mode; Connect with $q.screen for responsive behavior                                                                                                              |
| dark            | Boolean | false      | Notify the component that the background is a dark color                                                                                                                |
| flat            | Boolean | false      | Applies a 'flat' design (no default shadow)                                                                                                                             |
| bordered        | Boolean | false      | Applies a default border to the component                                                                                                                               |
| square          | Boolean | false      | Removes border-radius so borders are squared                                                                                                                            |
| loading         | Boolean | false      | Put Table into 'loading' state; Notify the user something is happening behind the covers                                                                                |
| no-data-label   | String  |            | Override default text to display when no data is available                                                                                                              |