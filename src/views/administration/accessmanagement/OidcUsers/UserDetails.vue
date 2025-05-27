<template>
  <div class="expanded-row tab-view">
    <b-tabs pills content-class="mt-3" style="height: 100%">
      <b-tab title="Membership">
        <b-container fluid class="p-0" style="max-width: 31.25rem">
          <b-form-group>
            <div class="list-group">
              <span v-for="team in teams" :key="team.name">
                <actionable-list-group-item
                  :value="team.name"
                  :delete-icon="true"
                  v-on:actionClicked="removeTeamMembership(team.uuid)"
                />
              </span>
              <actionable-list-group-item
                :add-icon="true"
                v-on:actionClicked="
                  $root.$emit('bv::show::modal', 'selectTeamModal')
                "
              />
            </div>
          </b-form-group>
        </b-container>
      </b-tab>
      <b-tab :title="$t('admin.permissions')">
        <b-container fluid class="p-0" style="max-width: 31.25rem">
          <b-form-group>
            <div class="list-group">
              <span v-for="permission in permissions" :key="permission.name">
                <actionable-list-group-item
                  :value="permission.name"
                  :delete-icon="true"
                  v-on:actionClicked="removePermission(permission)"
                />
              </span>
              <actionable-list-group-item
                :add-icon="true"
                v-on:actionClicked="
                  $root.$emit('bv::show::modal', 'selectPermissionModal')
                "
              />
            </div>
          </b-form-group>
        </b-container>
      </b-tab>
      <b-tab :title="this.$t('message.projects')">
        <div class="" style="width: 100%">
          <div v-if="loading" class="d-flex justify-content-center">
            <b-spinner variant="primary" type="grow" label="Loading"
              >Loading ...
            </b-spinner>
          </div>
          <div v-else>
            <label for="">{{ this.$t('message.projects') }}</label>
            <user-roles-table
              :parentContext="{ row, index }"
              :projectRoles="projectRoles"
              :availableRoles="availableRoles"
              @addProjectRole="addProjectRole"
              @updateProjectRole="updateProjectRole"
              @removeProjectRole="removeProjectRole"
            />
          </div>
        </div>
      </b-tab>

      <template #tabs-end>
        <li
          role="presentation"
          class="nav-item action-group"
          style="margin-left: auto; margin-top: auto"
        >
          <b-button
            style="height: 100%"
            variant="outline-danger"
            @click="deleteUser"
            >{{ $t('admin.delete_user') }}</b-button
          >
        </li>
      </template>
    </b-tabs>

    <b-row class="expanded-row p-3" colspan="2"> </b-row>
    <select-team-modal
      :currentTeams="teams"
      v-on:selection="updateTeamSelection"
    />
    <select-permission-modal
      :currentPermissions="permissions"
      v-on:selection="updatePermissionSelection"
    />
  </div>
</template>

<script>
import i18n from '../../../../i18n';
import permissionsMixin from '../../../../mixins/permissionsMixin';
import userManagementMixin from '../../../../mixins/userManagementMixin';
import ActionableListGroupItem from '../../../components/ActionableListGroupItem.vue';
import UserRolesTable from '../../../components/UserRolesTable.vue';
import SelectPermissionModal from '../SelectPermissionModal.vue';
import SelectTeamModal from '../SelectTeamModal.vue';

export default {
  i18n,
  props: {
    index: { type: Number, required: true },
    row: { type: Object, required: true },
    rowEvents: {
      update: { type: String },
      delete: { type: String },
      cacheKey: { type: String },
    },
  },
  mixins: [permissionsMixin, userManagementMixin],
  components: {
    ActionableListGroupItem,
    SelectTeamModal,
    SelectPermissionModal,
    UserRolesTable,
  },
  data() {
    return {
      user: this.row, // local cache
      username: this.row.username,
      teams: this.row.teams,
      permissions: this.row.permissions,
      projectRoles: null,
      availableRoles: null,
      loading: true,
      userType: 'oidc',
    };
  },
  beforeMount() {
    this.initFromSessionCache();
  },
  mounted() {
    this.loadUserManagementData();
  },
  watch: {
    user: {
      handler(newValue) {
        this.username = newValue.username;
        this.teams = newValue.teams;
        this.permissions = newValue.permissions;
      },
      deep: true,
    },
  },
  methods: {
    deleteUser: function () {
      const endpoint = `${this.$api.BASE_URL}/${this.$api.URL_USER_OIDC}`;
      this._deleteUser(endpoint);
    },

    updateTeamSelection: function (selections) {
      this._updateTeamSelection(selections);
    },

    removeTeamMembership: function (teamUUID) {
      this._removeTeamMembership(teamUUID);
    },

    updatePermissionSelection: function (selections) {
      this._updatePermissionSelection(selections);
    },

    removePermission: function (permission) {
      this._removePermission(permission);
    },

    addProjectRole: function (projectRole, callbacks) {
      this._handleProjectRole('add', projectRole, callbacks).then(async () => {
        this.projectRoles = await this.loadUserProjects(this.username);
      });
    },

    updateProjectRole: function (projectRole, callbacks) {
      this._handleProjectRole('update', projectRole, callbacks);
    },

    removeProjectRole: function (projectRole, callbacks) {
      this._handleProjectRole('remove', projectRole, callbacks).then(
        async () => {
          this.projectRoles = await this.loadUserProjects(this.username);
        },
      );
    },
  },
};
</script>
