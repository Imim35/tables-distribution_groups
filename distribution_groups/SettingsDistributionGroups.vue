<template>
  <div class="page" id="settings-distribution-groups">
    <d-row>
      <d-col sm="8">

        <!-- Хлебные крошки -->
        <breadcrumbs title="Группы распределения" :links="[{ name: 'Главная', to: 'Home' }, { name: 'Настройки', to: 'Settings' }, { name: 'Заявки' }]"/>
      </d-col>
      <d-col sm="4" class="d-flex justify-content-end align-items-center">
        <d-button size="sm" @click="create">Создать</d-button>
      </d-col>
    </d-row>

    <!-- Таблица распределения групп -->
    <v-server-table ref="settings-distribution-groups" class="dataTables_wrapper hover" :columns="columns" :options="options" @row-click="rowClick">
      <div slot="id" slot-scope="props" class="p-0">
        {{ props.row.id}}
      </div>
      <div slot="name" slot-scope="props">
        {{ props.row.name}}
      </div>
      <div slot="branches" slot-scope="props" >
        <template v-if="props.row.branches.data.length" >
          <ul class="list-unstyled mb-0" v-for="prop in props" :key="prop.id">
            <li v-for="branches in prop" :key="branches.id">
              <span class="list-unstyled" v-for="branch in branches.data" :key="branch.id"><span class="branches" >{{ branch.name }}</span></span>
            </li>
          </ul>
        </template>
        <template v-else>-</template>
      </div>
    </v-server-table>
  </div>
</template>

<script>
  export default {
    name: "SettingsDistributionGroups",
    data() {
      return {
        columns: ['id', 'name', 'branches'],
        options: {
          perPage      : 10,
          perPageValues: [10, 20, 50, 100],
          skin         : 'table dataTable',
          sortIcon: {
            base: 'fas float-right mt-1 text-muted',
            up  : 'fa-caret-up',
            down: 'fa-caret-down',
          },
          pagination: {
            edge: true,
            nav: 'scroll',
          },
          searchable: ['id', 'number', 'branches'],
          sortable  : ['id', 'number', 'branches'],
          headings  : {
            id      : '№',
            name    : 'Имя',
            branches: 'Филиалы'
          },
          texts: {
            count            : "Отображение с {from} по {to}. Всего {count} групп|{count} групп|Одина группа",
            first            : 'Первая',
            last             : 'Последняя',
            filter           : "Filter:",
            filterPlaceholder: "Поиск",
            limit            : "Отображать по: ",
            page             : "Страница:",
            noResults        : "Группы не найдены",
            filterBy         : "Фильтрация по {column}",
            loading          : 'Загрузка...',
            defaultOption    : 'Выбрано {column}',
            columns          : 'Колонки'
          },
          requestFunction(data) {
            return this.$store.dispatch('requests/distributionGroups/index', {
              options: {
                params: {
                  search        : data.query,
                  search_columns: this.options.searchable,
                  limit         : data.limit,
                  ascending     : data.ascending,
                  page          : data.page,
                  byColumn      : data.byColumn,
                  order         : data.orderBy,
                  include       : 'branches'
                }
              }
            })
          },
          responseAdapter: (response) => {
            return { data: response.data, count: response.pagination.total_count }
          },
        },
      }
    },
    methods: {
      create() {
        let self = this
        this.$popup.show('DistributionGroupCreate', {}, {}, {
          created() {
            self.$refs['settings-distribution-groups'].refresh()
          }
        })
      },
      //Переход на страницу группы
      rowClick(props) {
        this.$router.push({ name: 'SettingsDistributionGroup', params: { id: props.row.id }})
      }
    }
  }
</script>
<style lang="scss">
  #settings-distribution-groups {
    .branches {
      background-color: #ececec;
      padding: 5px;
      border-radius: 3px;
      margin-right: 1em;
    }
  }

</style>
