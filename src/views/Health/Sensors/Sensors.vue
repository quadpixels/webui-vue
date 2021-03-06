<template>
  <b-container fluid="xl">
    <page-title />
    <b-row>
      <b-col sm="6" md="5" xl="4">
        <search
          :placeholder="$t('pageSensors.searchForSensors')"
          @changeSearch="onChangeSearchInput"
          @clearSearch="onClearSearchInput"
        />
      </b-col>
      <b-col sm="3" md="3" xl="2">
        <table-cell-count
          :filtered-items-count="filteredRows"
          :total-number-of-cells="allSensors.length"
        ></table-cell-count>
      </b-col>
      <b-col sm="3" md="4" xl="6" class="text-right">
        <table-filter :filters="tableFilters" @filterChange="onFilterChange" />
      </b-col>
    </b-row>
    <b-row>
      <b-col xl="12">
        <table-toolbar
          ref="toolbar"
          :selected-items-count="selectedRows.length"
          @clearSelected="clearSelectedRows($refs.table)"
        >
          <template v-slot:export>
            <table-toolbar-export
              :data="selectedRows"
              :file-name="exportFileNameByDate()"
            />
          </template>
        </table-toolbar>
        <b-table
          ref="table"
          responsive="md"
          selectable
          no-select-on-click
          sort-icon-left
          hover
          no-sort-reset
          sticky-header="75vh"
          sort-by="status"
          show-empty
          :no-border-collapse="true"
          :items="filteredSensors"
          :fields="fields"
          :sort-desc="true"
          :sort-compare="sortCompare"
          :filter="searchFilter"
          :empty-text="$t('global.table.emptyMessage')"
          :empty-filtered-text="$t('global.table.emptySearchMessage')"
          @filtered="onFiltered"
          @row-selected="onRowSelected($event, filteredSensors.length)"
        >
          <!-- Checkbox column -->
          <template v-slot:head(checkbox)>
            <b-form-checkbox
              v-model="tableHeaderCheckboxModel"
              :indeterminate="tableHeaderCheckboxIndeterminate"
              @change="onChangeHeaderCheckbox($refs.table)"
            />
          </template>
          <template v-slot:cell(checkbox)="row">
            <b-form-checkbox
              v-model="row.rowSelected"
              @change="toggleSelectRow($refs.table, row.index)"
            />
          </template>

          <template v-slot:cell(status)="{ value }">
            <status-icon :status="statusIcon(value)" />
            {{ value }}
          </template>
          <template v-slot:cell(currentValue)="data">
            {{ data.value }} {{ data.item.units }}
          </template>
          <template v-slot:cell(lowerCaution)="data">
            {{ data.value }} {{ data.item.units }}
          </template>
          <template v-slot:cell(upperCaution)="data">
            {{ data.value }} {{ data.item.units }}
          </template>
          <template v-slot:cell(lowerCritical)="data">
            {{ data.value }} {{ data.item.units }}
          </template>
          <template v-slot:cell(upperCritical)="data">
            {{ data.value }} {{ data.item.units }}
          </template>
        </b-table>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import PageTitle from '@/components/Global/PageTitle';
import Search from '@/components/Global/Search';
import StatusIcon from '@/components/Global/StatusIcon';
import TableFilter from '@/components/Global/TableFilter';
import TableToolbar from '@/components/Global/TableToolbar';
import TableToolbarExport from '@/components/Global/TableToolbarExport';
import TableCellCount from '@/components/Global/TableCellCount';

import BVTableSelectableMixin from '@/components/Mixins/BVTableSelectableMixin';
import LoadingBarMixin from '@/components/Mixins/LoadingBarMixin';
import TableFilterMixin from '@/components/Mixins/TableFilterMixin';
import TableDataFormatterMixin from '@/components/Mixins/TableDataFormatterMixin';
import TableSortMixin from '@/components/Mixins/TableSortMixin';
import SearchFilterMixin from '@/components/Mixins/SearchFilterMixin';

export default {
  name: 'Sensors',
  components: {
    PageTitle,
    Search,
    StatusIcon,
    TableCellCount,
    TableFilter,
    TableToolbar,
    TableToolbarExport
  },
  mixins: [
    TableFilterMixin,
    BVTableSelectableMixin,
    LoadingBarMixin,
    TableDataFormatterMixin,
    TableSortMixin,
    SearchFilterMixin
  ],
  data() {
    return {
      fields: [
        {
          key: 'checkbox',
          sortable: false,
          label: ''
        },
        {
          key: 'name',
          sortable: true,
          label: this.$t('pageSensors.table.name')
        },
        {
          key: 'status',
          sortable: true,
          label: this.$t('pageSensors.table.status')
        },
        {
          key: 'lowerCritical',
          formatter: this.tableFormatter,
          label: this.$t('pageSensors.table.lowerCritical')
        },
        {
          key: 'lowerCaution',
          formatter: this.tableFormatter,
          label: this.$t('pageSensors.table.lowerWarning')
        },

        {
          key: 'currentValue',
          formatter: this.tableFormatter,
          label: this.$t('pageSensors.table.currentValue')
        },
        {
          key: 'upperCaution',
          formatter: this.tableFormatter,
          label: this.$t('pageSensors.table.upperWarning')
        },
        {
          key: 'upperCritical',
          formatter: this.tableFormatter,
          label: this.$t('pageSensors.table.upperCritical')
        }
      ],
      tableFilters: [
        {
          key: 'status',
          label: this.$t('pageSensors.table.status'),
          values: ['OK', 'Warning', 'Critical']
        }
      ],
      activeFilters: [],
      searchTotalFilteredRows: 0
    };
  },
  computed: {
    allSensors() {
      return this.$store.getters['sensors/sensors'];
    },
    filteredRows() {
      return this.searchFilter
        ? this.searchTotalFilteredRows
        : this.filteredSensors.length;
    },
    filteredSensors() {
      return this.getFilteredTableData(this.allSensors, this.activeFilters);
    }
  },
  created() {
    this.startLoader();
    this.$store
      .dispatch('sensors/getAllSensors')
      .finally(() => this.endLoader());
  },
  beforeRouteLeave(to, from, next) {
    this.hideLoader();
    next();
  },
  methods: {
    sortCompare(a, b, key) {
      if (key === 'status') {
        return this.sortStatus(a, b, key);
      }
    },
    onFilterChange({ activeFilters }) {
      this.activeFilters = activeFilters;
    },
    onFiltered(filteredItems) {
      this.searchTotalFilteredRows = filteredItems.length;
    },
    onChangeSearchInput(event) {
      this.searchFilter = event;
    },
    exportFileNameByDate() {
      // Create export file name based on date
      let date = new Date();
      date =
        date.toISOString().slice(0, 10) +
        '_' +
        date
          .toString()
          .split(':')
          .join('-')
          .split(' ')[4];
      return this.$t('pageSensors.exportFilePrefix') + date;
    }
  }
};
</script>
