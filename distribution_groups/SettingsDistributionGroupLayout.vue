<template>
  <div class="page" id="settings-distribution-group-layout" ref="settings-distribution-group-layout" >
    <d-row>
      <d-col sm="8">

        <!-- Хлебные крошки -->
        <breadcrumbs title="Просмотр" :links="[{ name: 'Главная', to: 'Home' }, { name: 'Настройки', to: 'Settings' }, { name: 'Группы распределения', to: 'SettingsDistributionGroups' }]"/>
      </d-col>
    </d-row>

    <!-- Основной контент -->
    <d-row v-if="group">
      <d-col lg="3" sm="12">
        <settings-distribution-group-card :group="group"/>
      </d-col>
      <d-col lg="9" sm="12">
        <router-view :group="group"/>
      </d-col>
    </d-row>
  </div>
</template>

<script>
  import store from '@/store'
  import SettingsDistributionGroupCard from './SettingsDistributionGroupCard';
  export default {
    name: 'SettingsDistributionGroupLayout',
    components: {  SettingsDistributionGroupCard },
    data() {
      return {
        group: null,
      }
    },
    // Получение группы распределения оп id
    async beforeRouteEnter(to, from, next) {
      try {
        if (!to.params.id) throw new Error('Не передан идентификатор группы распределения');
        let id = parseInt(to.params.id)

        let group = await store.dispatch('requests/distributionGroups/get', { id: id, options: { params: { include: 'branches,users' }} })

        next(vm => { vm.group = group  })
      } catch (e) {
        this.$notification.show('error', e.message)
      }
    },
  }
</script>

