<template>
  <q-table
    :data="getFilteredValuesData"
    :columns="columns"
    row-key="id"
    ref="table"
    :class="classes"
    :visible-columns="visibleColumns"
    :pagination.sync="paginationTable"
    :separator="separator"
    :dense="dense"
    :loading="loading"
    :dark="dark"
    :flat="flat"
    :bordered="bordered"
    :square="square"
    :filter="filter"
    :selected.sync="selected"
    :selection="selection"
    :no-data-label="noDataLabel"
  >
    <template v-slot:header="props">
      <q-tr
        :class="tableHeaderClass" 
        :props="props"
      >
        <q-th v-if="selection !== 'none'"/>
        <q-th
          :props="props"
          @hover.native.stop
          v-for="col in props.cols"
          :key="col.name"
          class="text-center"
        >
          <div class="row inline">
            <q-btn
              v-if="col.sortable === true"
              @click="reverseSortOptions(col)"
              :label="col.label"
              flat
              dense
              no-caps
              size="12px"
            />
            <span v-else>{{ col.label }}</span>

            <div v-if="headerFilter">
              <q-btn
                flat
                dense
                size="sm"
                icon="fa fa-filter"
                class="q-ml-xs"
                @click.stop
              >
                <q-icon
                  name="fas fa-asterisk"
                  color="red"
                  style="font-size: 7px"
                  v-if="column_options_selected[col.name].length > 0"
                ></q-icon>
                <q-menu>
                  <q-space />

                  <q-btn
                    dense
                    class="float-right q-ma-sm bg-red text-white"
                    round
                    size="sm"
                    v-close-popup
                    flat
                    icon="close"
                  />

                  <div class="q-pa-sm q-mt-md">
                    <q-select
                      map-options
                      multiple
                      emit-value
                      filled
                      v-model="column_options_selected[col.name]"
                      :options="column_options[col.name]"
                      style="width: 150px !important"
                    />
                  </div>
                  <q-btn
                    color="primary"
                    class="float-right q-mr-sm q-mb-sm text-capitalize"
                    size="sm"
                    v-close-popup
                    @click="$set(column_options_selected, col.name, [])"
                    label="Clear"
                  />
                </q-menu>
              </q-btn>
            </div>
          </div>
        </q-th>
      </q-tr>
      <q-tr :props="props" class="ignore-elements" v-if="columnsFilter">
        <q-th v-if="selection !== 'none'"/>
        <q-th
          :key="col.name"
          v-for="col in props.cols"
          :style="tableFilterStyle || 'padding: 0 5px'"
        >
          <div v-if="!col.disableFilter">
            <q-input
              v-if="
                !col.hasOwnProperty('filter_type') || col.filter_type == 'text'
              "
              dense
              color="teal"
              filled
              v-model="filter_data[col.name]"
            >
              <template v-if="filter_data[col.name]" v-slot:append>
                <q-icon
                  name="cancel"
                  @click.stop="filter_data[col.name] = ''"
                  class="cursor-pointer"
                />
              </template>
            </q-input>

            <q-select
              v-if="
                col.hasOwnProperty('filter_type') && col.filter_type == 'select'
              "
              map-options
              multiple
              emit-value
              filled
              v-model="column_options_selected[col.name]"
              :options="column_options[col.name]"
              dense
            >
              <template v-slot:append>
                <q-icon
                  v-if="column_options_selected[col.name].length > 0"
                  name="close"
                  @click.stop="$set(column_options_selected, col.name, [])"
                  class="cursor-pointer"
                />
              </template>
            </q-select>
          </div>
        </q-th>
      </q-tr>
    </template>

    <template
      v-for="propSlot in Object.keys($scopedSlots)"
      v-slot:[propSlot]="props"
    >
      <slot :name="propSlot" v-bind="props" />
    </template>
  </q-table>
</template>

<script>
import { getTypeOf } from '../utils'

export default {
  name: 'TableFilter',

  props: {
    data: {
      type: Array,
      default: null
    },
    pagination: {
      type: Object,
      default: null
    },
    visibleColumns: {
      type: Array,
      default: () => undefined
    },
    columns: {
      type: Array,
      default: null
    },
    filter: {
      type: String,
      default: ''
    },
    selection: {
      type: String,
      default: 'none'
    },
    classes: {
      type: String,
      default: ''
    },
    separator: {
      type: String,
      default: 'horizontal'
    },
    noDataLabel: {
      type: String,
      default: ''
    },

    tableHeaderClass: {
      type: String,
      default: ''
    },

    tableFilterStyle: {
      type: String,
      default: ''
    },

    columnsFilter: Boolean,
    headerFilter: Boolean,
    dense: Boolean,
    dark: Boolean,
    flat: Boolean,
    bordered: Boolean,
    square: Boolean,
    loading: Boolean
  },
  data () {
    return {
      filter_data: {},
      column_options: {},
      column_options_selected: {},
      paginationTable: {},
      selected: [],
      isMounted: false
    }
  },
  computed: {
    tableRef () {
      return this.isMounted ? this.$refs.table : {}
    },
    getFilteredData () {
      let self = this

      let tableColumns = this.columns.filter((c) => !c.disableFilter)

      let tableData = this.data.filter((item) => {
        for (let i = 0; i < tableColumns.length; i++) {
          if (self.filter_data[tableColumns[i].name] === '') continue

          const val = typeof tableColumns[i].field === 'function' ? tableColumns[i].field(item) : item[tableColumns[i].name]
          if (
            tableColumns[i].name in self.filter_data &&
            val
              .toString()
              .toLowerCase()
              .indexOf(self.filter_data[tableColumns[i].name].toLowerCase()) === -1
          ) {
            return false
          }
        }
        return true
      })

      return tableData
    },
    getFilteredValuesData () {
      let self = this

      let tableData = this.getFilteredData.filter((item) => {
        for (let i = 0; i < self.columns.length; i++) {
          if (self.column_options_selected[self.columns[i].name].length === 0) continue

          const val = typeof self.columns[i].field === 'function' ? self.columns[i].field(item) : item[self.columns[i].name]
          if (
            self.column_options_selected[self.columns[i].name].indexOf(
              val
            ) === -1
          ) {
            return false
          }
        }
        return true
      })

      return tableData
    }
  },

  watch: {
    data () {
      this.setup()
    }
  },
  created () {
    this.setup()
  },
  mounted() {
    this.isMounted = true
  },
  methods: {
    reverseSortOptions (column) {
      this.column_options[column.name] = this.column_options[column.name].reverse()
    },

    setup () {
      this.paginationTable = this.pagination
      let self = this
      self.column_options = {}
      self.columns.forEach(item => {
        self.column_options[item.name] = []
        self.$set(self.column_options_selected, item.name, [])
      })

      const isFunction = prop => getTypeOf(prop) === 'function'

      const columnsFields = this.columns.filter((c) => !c.disableFilter)
        .map(({ field, name }) => isFunction(field) ? name : field)
        .map((field) => ([field, []]))

      let emptyOptionsByColumnField = Object.fromEntries(columnsFields)
      emptyOptionsByColumnField = JSON.parse(JSON.stringify(emptyOptionsByColumnField))

      const optionsByColumnField = this.data.reduce((cols, row) => {
        for (const key in row) {
          if (cols !== undefined && cols.hasOwnProperty(key)) {
            const value = row[key]
            const column = this.columns.find(c => c.name === key)
            const hasInColumns = Boolean(cols[key].find(col => getTypeOf(value) === 'object'
              ? col.value === column.field(row) : col.value === value))

            if (hasInColumns === false) {
              const rowLabelValue = isFunction(column.field) ? column.field(row) : value
              
              cols[key].push({ label: rowLabelValue, value: rowLabelValue })
            }
          }
        }

        return cols
      }, emptyOptionsByColumnField)

      const sortedOptionsByColumnField = this.columns.reduce((colsOpts, column) => {
        if (!column.disableFilter && column.sortable === true) {
          const columnKey = isFunction(column.field) ? column.name : column.field

          const sortMethodType = isFunction(column.sort) ? column.sort : (a, b) => String(a).localeCompare(String(b))

          const sortedOptions = colsOpts[columnKey].sort((a, b) => sortMethodType(a.value, b.value))

          return {
            ...colsOpts,
            [columnKey]: sortedOptions
          }
        }

        return colsOpts
      }, optionsByColumnField)

      this.column_options = sortedOptionsByColumnField
    }
  }
}
</script>
