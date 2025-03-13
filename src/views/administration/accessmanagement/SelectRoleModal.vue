<template>
  <b-modal
    id="selectRoleModal"
    @hide="resetValues()"
    size="md"
    hide-header-close
    no-stacking
    :title="$t('admin.assign_role')"
  >
    <b-card no-body :header="header">
      <b-card-body class="pb-0">
        <b-input-group-form-select
          v-model="$selectedProject"
          :options="availableProjects"
          :label="$t('admin.project')"
          rules="required"
        />
      </b-card-body>
      <b-card-body class="pb-0">
        <b-input-group-form-select
          v-model="$selectedRole"
          :options="availableRoles"
          :label="$t('admin.role')"
          rules="required"
        />
      </b-card-body>
    </b-card>
    <template v-slot:modal-footer="{ cancel }">
      <b-button size="md" variant="secondary" @click="cancel()">{{
        $t('message.close')
      }}</b-button>
      <b-button size="md" variant="primary" @click="createRoleMapping()">{{
        $t('message.assign')
      }}</b-button>
    </template>
  </b-modal>
</template>

<script>
import BInputGroupFormSelect from '../../../forms/BInputGroupFormSelect';
import ActionableListGroupItem from '../../components/ActionableListGroupItem';
import SelectProjectModal from './SelectProjectModal';

export default {
  name: 'selectRoleModal',
  components: {
    BInputGroupFormSelect,
    ActionableListGroupItem,
    SelectProjectModal,
  },
  created() {
    this.initialRepositoryType = this.type;
    this.repositoryType = this.type;
  },
  mounted() {
    this.loadRoles();
    this.loadProjects();
  },
  props: {
    username: String,
  },
  data() {
    return {
      selectedProject: null,
      selectedRole: null,
      availableRoles: [],
      availableProjects: [],
      labelIcon: {
        dataOn: '\u2713',
        dataOff: '\u2715',
      },
    };
  },
  methods: {
    createRoleMapping: function () {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_USER}/${this.username}/role`;
      this.axios
        .post(url, {
          projectName: this.selectedProject.name,
          projectVersion: this.selectedProject.version,
          selectedRole: this.selectedRole.uuid,
        })
        .then((response) => {
          this.$emit('refreshTable');
          this.$toastr.s(this.$t('admin.repository_created'));
          this.$root.$emit(
            'bv::hide::modal',
            'repositoryCreateRepositoryModal',
          );
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
          this.$root.$emit(
            'bv::hide::modal',
            'repositoryCreateRepositoryModal',
          );
        });
      this.resetValues();
    },
    updateProjectSelection: function (selections) {
      this.$root.$emit('bv::hide::modal', 'selectProjectModal');
      for (let i = 0; i < selections.length; i++) {
        let selection = selections[i];
        let url = `${this.$api.BASE_URL}/${this.$api.URL_ACL_MAPPING}`;
        this.axios
          .put(url, {
            team: this.team.uuid,
            project: selection.uuid,
          })
          .then((response) => {
            if (this.projects === undefined || this.projects === null) {
              this.projects = [];
            }
            this.projects.push({
              name: selection.name,
              version: selection.version,
              uuid: selection.uuid,
            });
            this.projects.sort();
            this.$toastr.s(this.$t('message.updated'));
          })
          .catch((error) => {
            if (error.response.status === 304) {
              //this.$toastr.w(this.$t('condition.unsuccessful_action'));
            } else {
              this.$toastr.w(this.$t('condition.unsuccessful_action'));
            }
          });
      }
    },
    removeProjectMapping: function (projectUuid) {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_ACL_MAPPING}/team/${this.team.uuid}/project/${projectUuid}`;
      this.axios
        .delete(url)
        .then((response) => {
          let k = [];
          for (let i = 0; i < this.projects.length; i++) {
            if (this.projects[i].uuid !== projectUuid) {
              k.push(this.projects[i]);
            }
          }
          this.projects = k;
          this.$toastr.s(this.$t('message.updated'));
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
        });
    },
    loadRoles: function () {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_ROLE}`;
      this.axios
        .get(url)
        .then((response) => {
          this.availableRoles = response.data.map((d) => ({
            value: d.id,
            text: d.name,
          }));
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
        });
    },
    loadProjects: function () {
      let url = `${this.$api.BASE_URL}/${this.$api.URL_ACL_USER}/${this.username}`;
      this.axios
        .get(url)
        .then((response) => {
          this.availableProjects = response.data.map((d) => ({
            value: d.id,
            text: d.name,
          }));
          // this.availableProjects = response.data;
        })
        .catch((error) => {
          this.$toastr.w(this.$t('condition.unsuccessful_action'));
        });
    },
    resetValues: function () {
      this.repositoryType = this.initialRepositoryType;
      this.url = null;
    },
  },
};
</script>
