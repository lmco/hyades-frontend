<template>
  <b-modal
    id="selectRoleModal"
    size="md"
    @hide="resetValues()"
    hide-header-close
    no-stacking
    :title="$t('admin.select_role')"
  >
    <b-input-group :size="md" class="mb-3" prepend="Project">
      <b-button size="md" variant="primary" v-b-modal="`selectProjectModal`">{{
        $t('admin.select_project')
      }}</b-button>
      <b-form-input></b-form-input>
      <b-input-group-append prepend="Role">
        <b-dropdown text="Role" variant="info">
          <b-dropdown-item>Action A</b-dropdown-item>
          <b-dropdown-item>Action B</b-dropdown-item>
        </b-dropdown>
      </b-input-group-append>
    </b-input-group>

    <template v-slot:modal-footer="{ cancel }">
      <b-button size="md" variant="secondary" @click="cancel()">{{
        $t('message.close')
      }}</b-button>
      <b-button size="md" variant="primary" @click="createRole()">{{
        $t('message.create')
      }}</b-button>
    </template>
  </b-modal>
</template>

<script>
import permissionsMixin from '../../../mixins/permissionsMixin';
import SelectProjectModal from './SelectProjectModal.vue';

export default {
  mixins: [permissionsMixin],
  components: [SelectProjectModal],
  data() {
    return {
      role: role,
      project: project,
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
      data: [],
      options: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        silentSort: false,
        sidePagination: 'client',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        icons: {
          refresh: 'fa-refresh',
        },
        responseHandler: function (res, xhr) {
          res.total = xhr.getResponseHeader('X-Total-Count');
          return res;
        },
        url: `${this.$api.BASE_URL}/${this.$api.URL_ROLE}`,
      },
    };
  },
  methods: {
    selectRole(row, role) {
      // Handle role selection for the row
      row.role = role.name;
      this.$refs.table.refresh();
    },
  },
};
</script>
