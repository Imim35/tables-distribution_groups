<template>
  <div class="vld-parent" ref="settings-distribution-group-main">
    <d-card class="card-small mb-4">

      <!-- Заголовок -->
      <d-card-header class="border-bottom">
        <h5 class="mb-0">Основная информация</h5>
        <span class="text-muted d-block font-weight-light">Редактирование основных данных</span>
      </d-card-header>

      <!-- Контент -->
      <d-card-body>
        <d-form novalidate @submit.prevent="submit" id="group-form">
          <d-row class="form-group">
            <d-col sm="6">
              <label for="name">
                <strong class="d-block">Название группы распределения</strong>
              </label>
              <d-form-input :state="!$v.distributionGroup.name.$invalid" type="text" id="name" placeholder="Колл центр номер 4" v-model.trim="$v.distributionGroup.name.$model" required/>
            </d-col>
          </d-row>
          <d-row>
            <d-col sm="12">
              <d-button type="submit" class="btn-accent" :disabled="this.$v.$invalid">Сохранить</d-button>
            </d-col>
          </d-row>
        </d-form>
      </d-card-body>
    </d-card>
  </div>
</template>

<script>
  import { required } from "vuelidate/lib/validators";
  export default {
    name: "SettingsDistributionGroupMain",
    props: {
      group: {
        type    : Object,
        required: true
      }
    },
    data() {
      return {
        distributionGroup: null
      }
    },
    validations: {
      distributionGroup: {
        name : { required },
      }
    },
    // Добавление источника и региона в компонент
    mounted() {
      this.distributionGroup = this.group
    },
    methods:{
      submit() {
        // Анимация загрузки.
        let loader = this.$loader.show({ container: this.$refs['settings-distribution-group-main'] })

        let payload = { name : this.$v.distributionGroup.name.$model }

        // Обновление данных источника
        this.$store.dispatch('requests/distributionGroups/update', { id: this.group.id, payload })
          .then(() => {
            this.$notification.show('success', 'Группа успешно изменина')
            this.group.name = this.$v.distributionGroup.name.$model
            setTimeout(() => { this.$emit('close') }, 200)
          })
          .catch(e => {
            this.$notification.show('error', e.message)
          })
          .finally(() => { loader.hide() })
      }
    }
  }
</script>

