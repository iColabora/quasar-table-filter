<template>
  <q-table
    :data="getFilteredValuesData"
    :columns="columns"
    row-key="name"
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
  >
    <template v-slot:header="props">
      <q-tr :props="props">
        <q-th
          :props="props"
          @hover.native.stop
          v-for="col in props.cols"
          :key="col.name"
        >
          <div class="row inline">
            <div class="column">
              <p>{{ col.label }}</p>
            </div>
            <div class="column">
              <q-btn
                flat
                dense
                size="sm"
                icon="fa fa-filter"
                class="q-ml-xs"
                @click.stop
                v-if="headerFilter"
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
        <q-th
          :key="col.name"
          v-for="col in props.cols"
          style="padding: 0px 0px 0px 0px"
        >
          <div v-if="!col.disableFilter">
            <q-input
              v-if="
                !col.hasOwnProperty('filter_type') || col.filter_type == 'text'
              "
              dense
              color="teal"
              class="q-pl-xs q-pr-xs"
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
      v-slot:top-right="props"
      v-if="excelDownload || csvDownload || fullscreen || globalSearch"
    >
      <q-input
        filled
        v-if="globalSearch"
        borderless
        dense
        debounce="300"
        v-model="filter"
        class="q-mr-md"
        :placeholder="searchPlaceholder"
      >
        <template v-slot:append>
          <q-icon name="search" />
        </template>
      </q-input>

      <q-btn
        class="bg-grey-2 q-mr-sm"
        icon="fas fa-file-excel"
        no-caps
        v-if="excelDownload"
        :label="excelDownloadLabel"
        @click="exportTable('xlsx')"
      />

      <q-btn
        class="bg-primary text-white"
        icon="fas fa-file-csv"
        no-caps
        v-if="csvDownload"
        :label="csvDownloadLabel"
        @click="exportTable('csv')"
      />

      <q-btn
        v-if="fullscreen"
        flat
        round
        class="q-ml-sm"
        dense
        :icon="props.inFullscreen ? 'fullscreen_exit' : 'fullscreen'"
        @click="props.toggleFullscreen"
      >
        <q-tooltip :disable="$q.platform.is.mobile" v-close-popup>{{
          props.inFullscreen ? "Exit Fullscreen" : "Toggle Fullscreen"
        }}</q-tooltip>
      </q-btn>
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
import { exportFile } from 'quasar'

function wrapCsvValue (val, formatFn) {
  let formatted = formatFn !== void 0 ? formatFn(val) : val

  formatted =
    formatted === void 0 || formatted === null ? '' : String(formatted)

  formatted = formatted.split('"').join('""')

  return `"${formatted}"`
}

export default {
  name: 'TableFilter',

  props: {
    data: {
      type: [Array, Object],
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
      type: [Array, Object],
      default: null
    },
    fileName: {
      type: String,
      default: 'Download'
    },

    classes: {
      type: String,
      default: ''
    },
    separator: {
      type: String,
      default: 'horizontal'
    },

    searchPlaceholder: {
      type: String,
      default: 'Search'
    },

    csvDownloadLabel: {
      type: String,
      default: ''
    },

    excelDownloadLabel: {
      type: String,
      default: ''
    },

    csvDownload: Boolean,
    excelDownload: Boolean,
    columnsFilter: Boolean,
    headerFilter: Boolean,
    dense: Boolean,
    dark: Boolean,
    flat: Boolean,
    bordered: Boolean,
    square: Boolean,
    globalSearch: Boolean,
    fullscreen: Boolean,
    loading: Boolean
  },
  data () {
    return {
      filter_data: {},
      column_options: {},
      column_options_selected: {},
      filter: '',
      paginationTable: {}
    }
  },
  computed: {
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
  methods: {
    setup () {
      this.paginationTable = this.pagination
      let self = this
      self.column_options = {}
      self.columns.forEach(item => {
        self.column_options[item.name] = []
        self.$set(self.column_options_selected, item.name, [])
      })

      const columnsFields = this.columns.filter((c) => !c.disableFilter)
        .map(({ field, name }) => typeof field === 'function' ? name : field)
        .map((field) => ([field, []]))

      const columns = Object.fromEntries(columnsFields)

      const options = this.data.reduce((cols, row) => {
        for (const key in row) {
          if (cols !== undefined && cols.hasOwnProperty(key)) {
            const value = row[key]
            const column = this.columns.find(c => c.name === key)
            const hasInColumns = Boolean(cols[key].find(col => typeof value === 'object'
              ? col.value === column.field(row) : col.value === value))

            if (hasInColumns === false) {
              if (typeof column.field === 'function') {
                const val = column.field(row)
                cols[key].push({ label: val, value: val })
              } else {
                cols[key].push({ label: value, value })
              }
            }
          }
        }

        return cols
      }, JSON.parse(JSON.stringify(columns)))

      this.column_options = options
    },
    exportTable (type) {
      // naive encoding to csv format
      const content = [this.columns.filter((c) => !c.disableFilter).map((col) => wrapCsvValue(col.label))]
        .concat(
          this.data.map((row) =>
            this.columns.filter((c) => !c.disableFilter)
              .map((col) =>
                wrapCsvValue(
                  typeof col.field === 'function'
                    ? col.field(row)
                    : row[col.field === void 0 ? col.name : col.field],
                  col.format
                )
              )
              .join(',')
          )
        )
        .join('\r\n')

      const status = exportFile(
        this.fileName + '.' + type,
        content,
        'text/' + type
      )

      if (status !== true) {
        this.$q.notify({
          message: 'Browser denied file download...',
          color: 'negative',
          icon: 'warning'
        })
      }
    }
  }
}
</script>

<style scoped>
</style>
