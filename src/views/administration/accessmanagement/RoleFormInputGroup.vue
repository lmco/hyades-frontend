<template>
  <bootstrap-table
    ref="table"
    :columns="columns"
    :data="data"
    :options="options"
  >
  </bootstrap-table>
</template>

<script>
import xssFilters from 'xss-filters';
import common from '../../../shared/common';
import bootstrapTableMixin from '../../../mixins/bootstrapTableMixin';
import EventBus from '../../../shared/eventbus';
import BInputGroupFormSelect from '../../../forms/BInputGroupFormSelect';

export default {
  props: {
    header: String,
  },
  mixins: [bootstrapTableMixin],
  components: {
    BInputGroupFormSelect,
  },
  mounted() {
    EventBus.$on('admin:roles:rowUpdate', (index, row) => {
      this.$refs.table.updateRow({ index: index, row: row });
      this.$refs.table.expandRow(index);
    });
    EventBus.$on('admin:roles:rowDeleted', (index, row) => {
      this.refreshTable();
    });
  },
  beforeDestroy() {
    EventBus.$off('admin:roles:rowUpdate');
    EventBus.$off('admin:roles:rowDeleted');
  },
  data() {
    return {
      columns: [
        {
          title: this.$t('admin.project'),
          field: 'name',
          sortable: false,
          formatter(value, row, index) {
            return xssFilters.inHTMLData(common.valueWithDefault(value, ''));
          },
        },
        {
          title: this.$t('admin.role'),
          field: 'permissions',
          sortable: false,
          formatter(value, row, index) {
            return `
              <b-input-group-form-select
                v-model="row.role"
                :options="row.permissions.map(role => ({ value: role.id, text: role.name }))"
                :label="$t('admin.role')"
              />
            `;
          },
        },
      ],
      data: [],
      options: {
        responseHandler: function (res, xhr) {
          res.total = xhr.getResponseHeader('X-Total-Count');
          return res;
        },
        url: `${this.$api.BASE_URL}/${this.$api.URL_ROLE}`,
      },
    };
  },
  methods: {
    refreshTable: function () {
      this.$refs.table.refresh({
        silent: true,
      });
    },
  },
};
</script>
