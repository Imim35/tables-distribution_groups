<template>
  <div id="settings-call-center-groups-branches" class="vld-parent" ref="settings-call-center-groups-branches">
    <d-card class="card-small mb-4 ">

      <!-- Заголовок -->
      <d-card-header>
        <d-row>
          <d-col sm="8">
            <h5 class="mb-0">Филиалы</h5>
            <span class="text-muted d-block font-weight-light">Удаление / добавление филиалов  в группу</span>
          </d-col>

          <!-- Добавление филиалов -->
          <d-col sm="4">
            <d-form novalidate @submit.prevent="submit">
              <d-form-row>
                <d-input-group style="justify-content: flex-end">

                  <!-- Отображение филиалов по группам которые возможно добавить -->
                  <d-input-group-addon style="width: 80%;" prepend>
                    <d-form-select v-model="$v.form.branch_id.$model" :value="null" id="branch_id" style="border-top-right-radius: 0; border-bottom-right-radius: 0">
                      <option :value="null" selected>Выбрать...</option>
                      <option v-for="branch in branches" :key="branch.id" :value="branch.id">{{ branch.name }}</option>
                    </d-form-select>
                  </d-input-group-addon >

                  <!-- Добавление -->
                  <d-input-group-addon append>
                    <d-button type="submit" :disabled="$v.form.branch_id.$invalid">+</d-button>
                  </d-input-group-addon>
                </d-input-group>
              </d-form-row>
            </d-form>
          </d-col>
        </d-row>
      </d-card-header>

      <!-- Контент -->
      <d-card-body class="p-0">
        <d-row>
          <d-col sm="12">
            <v-server-table  ref="settings-call-center-groups-branches-table" id="settings-call-center-groups-branches-table" class="dataTables_wrapper hover" :columns="columns" :options="clientTableOptions">
              <div slot="id" slot-scope="props" class="p-0">
                {{ props.row.id }}
              </div>
              <div slot="name" slot-scope="props" class="name">
                {{ props.row.name }}
              </div>
              <div slot="actions" slot-scope="props" class="actions">
                <ul class="attachment-list list-unstyled mb-0">
                  <li class="attachment-item-actions-item" @click="detach(props.row.id)">
                    <i class="material-icons" style="font-size: 25px; color: #b50000" v-tooltip="'Открепить'" >cancel</i>
                  </li>
                </ul>
              </div>
            </v-server-table>
          </d-col>
        </d-row>
      </d-card-body>
    </d-card>
  </div>
</template>

<script>
  import { required } from "vuelidate/lib/validators";
  import store from "@/store";
  export default {
    name : "SettingsDistributionGroupBranches",
    props: {
      group: {
        type    : Object,
        required: true
      },
    },
    data() {
      return {
        state  : null,
        columns: ['id', 'name', 'actions'],
        loader: null,
        clientTableOptions: {
          filterable   : false,
          perPage      : 10,
          perPageValues: [],
          skin         : 'table dataTable',
          sortIcon: {
            base: 'fas float-right mt-1 text-muted',
            up  : 'fa-caret-up',
            down: 'fa-caret-down',
          },
          attributes: {
            group_id: this.group.id
          },
          pagination: {
            edge: true,
            nav : 'scroll'
          },
          searchable: ['id', 'name'],
          sortable  : ['id', 'name'],
          headings  : {
            'id'                : 'Id',
            'name'              : 'Название',
            'actions'           : 'Действия'
          },
          texts: {
            count             : "Отображение с {from} по {to}. Всего {count} филиалов|{count} филиалов|Один филиал",
            first             : 'Первая',
            last              : 'Последняя',
            filter            : "Filter:",
            filterPlaceholder : "Поиск",
            limit             : "Отображать по: ",
            page              : "Страница:",
            noResults         : "Филиалы не найдены",
            filterBy          : "Фильтрация по {column}",
            loading           : 'Загрузка...',
            defaultOption     : 'Выбрано {column}',
            columns           : 'Колонки'
          },
          requestFunction: (data) => {
            let payload = {
              id: this.clientTableOptions.attributes.group_id,
              options: {
                params: {
                  search        : data.query,
                  search_columns: this.clientTableOptions.searchable,
                  limit         : data.limit,
                  ascending     : data.ascending,
                  page          : data.page,
                  byColumn      : data.byColumn,
                  order         : data.orderBy
                }
              }
            };
            setTimeout(() => {
              this.loader = this.$loader.show({ container: this.$refs['settings-call-center-groups-branches'] })
            }, 0)
            return this.$store.dispatch('requests/distributionGroups/get_branches', payload)
              .finally(() => { this.loader.hide() })
          },
          responseAdapter: (response) => {
            this.exclude_ids = response.data.map(b => b.id)
            return { data: response.data, count: response.pagination.total_count }
          }
        },
        form: {
          branch_id: null
        },
        exclude_ids: []
      }
    },
    validations: {
      form: {
        branch_id : { required },
      },
    },

    // Получение всех филиалов системы
    async beforeRouteEnter(to, from, next) {
      try {
        await store.dispatch('branches/fetch')
        next()
      } catch (e) {
        this.$notification.show('error', e.message)
      }
    },
    computed: {

      // Отображение в select филиалов которых еще нет в группе
      branches() { return this.$store.state.branches.data.filter(b => { return this.exclude_ids.indexOf(b.id) === -1 })}
    },
    methods: {

      // Добавление филиала  в таблицу
      submit() {

        // Анимация загрузки.
        let loader = this.$loader.show({
          container: this.$refs['settings-call-center-groups-branches']
        })

        //Отправляемые данные
        let payload = { branch_id: this.$v.form.branch_id.$model }

        //Запрос на добавление филиала
        this.$store.dispatch('requests/distributionGroups/attach_branch', { id: this.group.id , payload })
          .then(() => {
            this.$refs['settings-call-center-groups-branches-table'].refresh()
            this.$notification.show('success', 'Филиал успешно добавлен')
            setTimeout(() => { this.form.branch_id = null }, 200)

          })
          .catch(e => { this.$notification.show('error', e.message) })
          .finally( () => { loader.hide() })
      },

      // Удаление филиала из таблицы
      detach(value) {

        // Анимация загрузки.
        let loader = this.$loader.show({
          container: this.$refs['settings-call-center-groups-branches']
        })
        // Отправляемые данные
        let payload = { branch_id: value }

        // Диалоговое окно подтверждения действия.
        this.$dialog.show('Вы действительно хотите открепить филиал от группы ?', { yes: 'Да', no: 'Нет'}, (confirm) => {
          if (!confirm) loader.hide()
          else {

            // Отправка запроса удаления.
            this.$store.dispatch('requests/distributionGroups/detach_branch', { id: this.group.id , payload })
              .then(() => {
                this.$refs['settings-call-center-groups-branches-table'].refresh()
                this.$notification.show('success', 'Филиал успешно откреплен')
              })
              .catch(e => { this.$notification.show('error', e.message) })
              .finally(() => { loader.hide() })
          }
        })
      }
    }
  }
</script>

<style lang="scss">
  #settings-call-center-groups-branches {
    ul.attachment-list {
      border-radius: 3px;
      padding: 4px 8px;

      li.attachment-item-actions-item {
        cursor: pointer;
        display: flex;
        justify-content: flex-end;
      }
    }
    .material-icons {
      transition: all 0.3s ease;
    }
    .material-icons:hover {
      color: #8F0000!important;
    }
  }
</style>
