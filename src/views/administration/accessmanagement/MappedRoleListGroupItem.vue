<template>
  <b-list-group-item class="flex-column align-items-start">
    <div class="d-flex w-100 justify-content-between">
      <span class="text-monospace">{{ apiKey.key }}</span>
      <div class="d-flex">
        <b-button
          size="sm"
          class="action-icon"
          v-b-tooltip.hover
          v-b-modal="`selectProjectModal`"
          :title="$t('admin.project')"
        >
          <span class="fa fa-edit"></span>
        </b-button>
        <b-button
          size="sm"
          class="action-icon ml-3"
          v-on:click="$emit('removeClicked')"
          v-b-tooltip.hover
          :title="$t('admin.remove_assigned_role')"
        >
          <span class="fa fa-trash-o"></span>
        </b-button>
        <edit-api-key-comment-modal
          :key-id="this.keyId"
          :api-key="this.apiKey"
        />
      </div>
    </div>
    <p class="mt-2 font-weight-light">
      <em>{{ comment }}</em>
    </p>
    <hr />
  </b-list-group-item>
</template>

<script>
import SelectProjectModal from './SelectProjectModal.vue';

export default {
  props: {
    role: Object,
    project: Object,
    variant: String,
    href: String,
  },
  components: {
    SelectProjectModal,
  },
  computed: {
    assigned_role: function () {
      return this.role;
    },
    assigned_project: function () {
      return this.project ? this.project.name : 'Not Assigned to a project';
    },
  },
};
</script>

<style lang="scss" scoped>
.list-group-item .form-group {
  padding-top: 0;
  padding-bottom: 0;
  margin-top: 0;
  margin-bottom: 0;
}

.action-icon {
  padding: 0;
  margin: 0;
  border: 0;
  background-color: transparent;
}

.action-icon .fa {
  font-size: 1.2rem;
}

.action-icon .fa-edit {
  color: var(--secondary);
}

.action-icon .fa-trash-o {
  color: var(--danger);
}

.list-group-item:last-child {
  margin-bottom: -1px;
}
</style>
