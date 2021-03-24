<template>
  <div id="settings-distribution-group-users" class="vld-parent" ref="settings-distribution-group-users">
    <d-card class="card-small mb-4">

      <!-- Заголовок -->
      <d-card-header>
        <d-row>
          <d-col sm="8">
            <h5 class="mb-0">Пользователи</h5>
            <span class="text-muted d-block font-weight-light">Удаление / добавление пользователей в группу</span>
          </d-col>

          <!-- Добавление пользователя -->
          <d-col sm="4">
            <d-form novalidate @submit.prevent="submit">
              <d-form-row>
                <d-input-group style="justify-content: flex-end">

                  <!-- Отображение пользователей по группам , не состоящие в группе -->
                  <d-input-group-addon style="width: 80%;" prepend>
                    <d-form-select v-model="$v.form.user_id.$model" :value="null" id="user_id" style="border-top-right-radius: 0; border-bottom-right-radius: 0">
                      <option :value="null" selected>Выбрать...</option>
                      <optgroup v-for="role in $store.getters['users/groupByRole'](users)" :key="role.id" :label="role.display_name">
                        <option v-for="user in  role.users.data" :key="user.id" :value="user.id">{{ user.username }}</option>
                      </optgroup>
                    </d-form-select>
                  </d-input-group-addon >

                  <!-- Добавление -->
                  <d-input-group-addon append>
                    <d-button type="submit" :disabled="$v.form.user_id.$invalid">+</d-button>
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
            <v-server-table ref="settings-distribution-group-users-table" id="settings-distribution-group-users-table" class="dataTables_wrapper hover" :columns="columns" :options="clientTableOptions">
              <div slot="id" slot-scope="props" class="p-0">
                {{ props.row.id }}
              </div>
              <div slot="name" slot-scope="props" class="name">
                <small class="d-block text-muted">{{ props.row.role.display_name }}</small>
                <span class="mr-1">{{ props.row.username }}</span>
                <span  v-if="props.row.lastname">{{ props.row.lastname }}</span>
              </div>
              <div slot="actions" slot-scope="props" class="actions">
                <ul class="attachment-list list-unstyled mb-0">
                  <li class="attachment-item-actions-item" @click="detach({ id: props.row.id, name: props.row.username })">
                    <i class="material-icons" style="font-size: 25px; color: #b50000" v-tooltip="'Открепить'">cancel</i>
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
  import store from "@/store";
  import { required } from "vuelidate/lib/validators";
  export default {
    name : "SettingsDistributionGroupUsers",
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
            'name'              : 'ФИО',
            'actions'           : 'Действия'
          },
          texts: {
            count             : "Отображение с {from} по {to}. Всего {count} пользователей|{count} пользователей|Один пользователь",
            first             : 'Первая',
            last              : 'Последняя',
            filter            : "Filter:",
            filterPlaceholder : "Поиск",
            limit             : "Отображать по: ",
            page              : "Страница:",
            noResults         : "Пользователи не найдены",
            filterBy          : "Фильтрация по {column}",
            loading           : 'Загрузка...',
            defaultOption     : 'Выбрано {column}',
            columns           : 'Колонки'
          },
          user_name: '',
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
              this.loader = this.$loader.show({ container: this.$refs['settings-distribution-group-users'] })
            }, 0)

            return this.$store.dispatch('requests/distributionGroups/getUsers', payload)
              .finally(() => { this.loader.hide() })
          },
          responseAdapter: (response) => {
            this.exclude_ids = response.data.map(b => b.id)
            return { data: response.data, count: response.pagination.total_count }
          }
        },
        form: {
          user_id: null
        },
        exclude_ids: []
      }
    },
    validations: {
      form: {
        user_id : { required },
      },
    },

    // Получение всех юзеров системы
    async beforeRouteEnter(to, from, next) {
      try {
        await store.dispatch('users/fetch', )
        next()
      } catch (e) {
        this.$notification.show('error', e.message)
      }
    },
    computed: {
      // Отображение в select пользователей которых еще нет в группе
      users() { return this.$store.state.users.data.filter(b => { return this.exclude_ids.indexOf(b.id) === -1 }) }
    },
    methods: {

      // Добавление пользователя  в таблицу
      submit() {

        // Анимация загрузки.
        let loader = this.$loader.show({
          container: this.$refs['settings-distribution-group-users']
        })

        //Отправляемые данные
        let payload = { user_id: this.$v.form.user_id.$model }

        //Отправка запроса на добавление
        this.$store.dispatch('requests/distributionGroups/attach_user', { id: this.group.id , payload })
          .then(() => {
            this.$refs['settings-distribution-group-users-table'].refresh()
            this.$notification.show('success', 'Пользователь успешно добавлен')
            setTimeout(() => { this.form.user_id = null  }, 200)
          })
          .catch(e => { this.$notification.show('error', e.message) })
          .finally( () => { loader.hide() })
      },

      // Удаление пользователя  из таблицы
      detach(value) {

        // Анимация загрузки.
        let loader = this.$loader.show({
          container: this.$refs['settings-distribution-group-users']
        })

        //Отправляемые данные
        let payload = { user_id: value.id }

        // Диалоговое окно подтверждения действия.
        this.$dialog.show(`Вы действительно хотите открепить пользователя от группы распределения ${ value.name } ?`, { yes: 'Да', no: 'Нет'}, (confirm) => {
          if (!confirm) loader.hide()
          else {
            // Отправка запроса удаления.
            this.$store.dispatch('requests/distributionGroups/detach_user', { id: this.group.id , payload })
              .then(() => {
                this.$refs['settings-distribution-group-users-table'].refresh()
                this.$notification.show('success', 'Пользователь успешно откреплен')
              })
              .catch(e => { this.$notification.show('error', e.message)})
              .finally(() => { loader.hide() })
          }
        })
      },
    }
  }
</script>

<style lang="scss">
  #settings-distribution-group-users {
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
