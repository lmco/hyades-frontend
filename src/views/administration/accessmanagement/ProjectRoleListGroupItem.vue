<template>
  <b-list-group-item class="flex-column align-items-start">
    <div class="d-flex w-100 justify-content-between">
      <span class="text-monospace"
        >{{ projectRole.role.name }} ({{ projectRole.project.name }})</span
      >
      <div class="d-flex">
        <b-button
          size="sm"
          class="action-icon ml-3"
          v-on:click="$emit('removeClicked')"
          v-b-tooltip.hover
          :title="$t('admin.remove_assigned_role')"
        >
          <span class="fa fa-trash-o"></span>
        </b-button>
      </div>
    </div>
  </b-list-group-item>
</template>

<script>
export default {
  props: {
    projectRole: {
      role: {
        name: '',
        uuid: '',
      },
      project: {
        name: '',
        uuid: '',
      },
    },
  },
};
</script>

<style lang="scss" scoped>
.action-icon {
  padding: 0;
  margin: 0;
  border: 0;
  background-color: transparent;
}

.action-icon .fa-trash-o {
  color: var(--danger);
}

.list-group-item:last-child {
  margin-bottom: -1px;
}
</style>
