<template>
  <page-section :section-title="$t('pageHardwareStatus.fans')">
    <b-row>
      <b-col sm="6" md="5" xl="4">
        <search
          @changeSearch="onChangeSearchInput"
          @clearSearch="onClearSearchInput"
        />
      </b-col>
      <b-col sm="6" md="3" xl="2">
        <table-cell-count
          :filtered-items-count="filteredRows"
          :total-number-of-cells="fans.length"
        ></table-cell-count>
      </b-col>
    </b-row>
    <b-table
      sort-icon-left
      no-sort-reset
      hover
      responsive="md"
      sort-by="health"
      show-empty
      :items="fans"
      :fields="fields"
      :sort-desc="true"
      :sort-compare="sortCompare"
      :filter="searchFilter"
      :empty-text="$t('global.table.emptyMessage')"
      :empty-filtered-text="$t('global.table.emptySearchMessage')"
      @filtered="onFiltered"
    >
      <!-- Expand chevron icon -->
      <template v-slot:cell(expandRow)="row">
        <b-button
          variant="link"
          data-test-id="hardwareStatus-button-expandFans"
          :aria-label="expandRowLabel"
          @click="toggleRowDetails(row)"
        >
          <icon-chevron :title="expandRowLabel" />
        </b-button>
      </template>

      <!-- Health -->
      <template v-slot:cell(health)="{ value }">
        <status-icon :status="statusIcon(value)" />
        {{ value }}
      </template>

      <template v-slot:row-details="{ item }">
        <b-container fluid>
          <b-row>
            <b-col sm="6" xl="4">
              <dl>
                <!-- Status state -->
                <dt>{{ $t('pageHardwareStatus.table.statusState') }}:</dt>
                <dd>{{ tableFormatter(item.statusState) }}</dd>
              </dl>
            </b-col>
          </b-row>
        </b-container>
      </template>
    </b-table>
  </page-section>
</template>

<script>
import PageSection from '@/components/Global/PageSection';
import IconChevron from '@carbon/icons-vue/es/chevron--down/20';
import TableCellCount from '@/components/Global/TableCellCount';

import StatusIcon from '@/components/Global/StatusIcon';
import TableDataFormatterMixin from '@/components/Mixins/TableDataFormatterMixin';
import TableSortMixin from '@/components/Mixins/TableSortMixin';
import Search from '@/components/Global/Search';
import SearchFilterMixin from '@/components/Mixins/SearchFilterMixin';
import TableRowExpandMixin from '@/components/Mixins/TableRowExpandMixin';

export default {
  components: { IconChevron, PageSection, StatusIcon, Search, TableCellCount },
  mixins: [
    TableRowExpandMixin,
    TableDataFormatterMixin,
    TableSortMixin,
    SearchFilterMixin
  ],
  data() {
    return {
      fields: [
        {
          key: 'expandRow',
          label: '',
          tdClass: 'table-row-expand',
          sortable: false
        },
        {
          key: 'id',
          label: this.$t('pageHardwareStatus.table.id'),
          formatter: this.tableFormatter,
          sortable: true
        },
        {
          key: 'health',
          label: this.$t('pageHardwareStatus.table.health'),
          formatter: this.tableFormatter,
          sortable: true
        },
        {
          key: 'partNumber',
          label: this.$t('pageHardwareStatus.table.partNumber'),
          formatter: this.tableFormatter,
          sortable: true
        },
        {
          key: 'serialNumber',
          label: this.$t('pageHardwareStatus.table.serialNumber'),
          formatter: this.tableFormatter,
          sortable: true
        }
      ],
      searchTotalFilteredRows: 0
    };
  },
  computed: {
    filteredRows() {
      return this.searchFilter
        ? this.searchTotalFilteredRows
        : this.fans.length;
    },
    fans() {
      return this.$store.getters['fan/fans'];
    }
  },
  created() {
    this.$store.dispatch('fan/getFanInfo').finally(() => {
      // Emit initial data fetch complete to parent component
      this.$root.$emit('hardwareStatus::fans::complete');
    });
  },
  methods: {
    sortCompare(a, b, key) {
      if (key === 'health') {
        return this.sortStatus(a, b, key);
      }
    },
    onFiltered(filteredItems) {
      this.searchTotalFilteredRows = filteredItems.length;
    }
  }
};
</script>
