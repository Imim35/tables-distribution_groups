<template>
  <d-card class="settings-distribution-group-card">
    <!--Заголовок-->
    <d-card-header class=" p-3">
      <d-row class="form-group">
        <d-col>
          <span class="text-muted d-block font-weight-light">Название</span>
          <h5 class="mb-0">{{ group.name }}</h5>
        </d-col>
      </d-row>
      <template  v-if="group.branches.data.length">
        <d-row class="form-group">
          <d-col>
            <span class="text-muted d-block font-weight-light">Филиалы</span>
            <ul v-for="branch in group.branches.data" :key="branch.id" class="mb-0 list-unstyled">
              <li>
                {{ branch.name }}
              </li>
            </ul>
          </d-col>
        </d-row>
      </template>
      <template v-if="group.users.data.length">
        <d-row>
          <d-col>
            <span class="text-muted d-block font-weight-light">Пользователи</span>
            <ul v-for="user in group.users.data" :key="user.id" class="mb-0 list-unstyled">
              <li>
                {{ user.username }}
              </li>
            </ul>
          </d-col>
        </d-row>
      </template>

    </d-card-header>

    <!--Контент-->
    <div class="menu">
      <d-list-group>
        <d-list-group-item v-for="(section, idx) in sections" style="cursor: pointer" :key="idx" class="p-0 pointer" :class="{ active: section.name === sectionName }">
          <div @click="change(section)" class="d-block px-3 py-2">
            <div>{{ section.name }}</div>
            <small class="text-muted d-block font-weight-light">{{ section.description }}</small>
          </div>
        </d-list-group-item>
      </d-list-group>
    </div>
  </d-card>
</template>

<script>
  export default {
    name: "SettingsDistributionGroupCard",
    props: {
      group: {
        type    : Object,
        required: true
      }
    },
    data() {
      return {
        sectionName:'Основное',
        sections: [
          { name: 'Основное', description: 'Редактирование основных данных', to: 'SettingsDistributionGroupMain' },
          { name: 'Пользователи', description: 'Управление пользователями группы распределения', to: 'SettingsDistributionGroupUsers' },
          { name: 'Филиалы', description: 'Управление филиалами группы распределения', to: 'SettingsDistributionGroupBranches' },
        ]
      }
    },
    methods: {
      change(section) {
        this.sectionName = section.name
        this.$emit('change', section.to)
      }
    }
  }
</script>

<style lang="scss">
  .settings-distribution-group-card {
    .details {
      .form-group-title {
        font-weight: 400;
        color: #3D5170;
        margin: 0;

        p {
          display: block;
        }
      }
      .form-group-value {
        display: block;
        color: #818EA3;
      }
    }
    .menu {
      .list-group-item {
        background-color: #f9f9f9;
        border-color: #e6e6e6;

        a {
          font-weight: 400; color: #000;
          small { color: #404040; }
        }
        &:first-child { border-radius: 0; }
        &.active {
          background-color: #007bff;
          border-color: #007bff;

          a, a small {
            color: #fff !important;
          }
        }
      }
    }
    .actions {
      a {
        font-weight: 400; color: #000;
        span { font-size: 1rem; }
        &.active {
          span { color: #007bff; }
        }
      }
    }
  }
</style>
