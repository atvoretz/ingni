<template>
  <q-page class="text-body2">
    <q-table
      title="Аккаунты клиентов"
      ref="tableRef"
      :rows="rows"
      :columns="columns"
      row-key="id"
      v-model:pagination="pagination"
      :loading="loading"
      :filter="filter"
      @request="onRequest"
      binary-state-sort
    >
      <template v-slot:header="props">
        <q-tr :props="props">
          <q-th v-for="col in props.cols" :key="col.name" :props="props">
            <b>{{ col.label }}</b>
          </q-th>
        </q-tr>
      </template>
      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td key="id" :props="props">
            {{ props.row.id }}
          </q-td>
          <q-td key="account_id" :props="props">
            {{ props.row.account_id }}
          </q-td>
          <q-td key="account_name" :props="props">
            <div class="q-pa-md q-gutter-sm">
              {{ props.row.account_name }}
              <q-badge color="info" transparent>
                v{{ props.row.amo_version }}</q-badge
              >

              <q-badge
                v-if="props.row.sipuni_account_id !== null"
                color="blue"
                transparent
              >
                Sipuni</q-badge
              >

              <q-badge
                color="black"
                v-if="props.row.sipuni_secret !== null"
                transparent
              >
                Готово для индикатора</q-badge
              >
            </div>
          </q-td>
          <q-td key="last_auth_date" :props="props">
            {{ props.row.last_auth_date }}
          </q-td>
          <q-td key="last_error_date" :props="props">
            {{ props.row.last_auth_date }}
          </q-td>
          <q-td key="expires" :props="props">
            {{ props.row.last_auth_date }}
          </q-td>
        </q-tr>
      </template>
      <template v-slot:top-right>
        <q-input
          borderless
          dense
          debounce="300"
          v-model="filter"
          placeholder="Поиск"
        >
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
      </template>
    </q-table>
  </q-page>
</template>

<script>
import JsonViewer from 'vue-json-viewer';
import { ref, onMounted } from 'vue';
import { defineComponent } from 'vue';
import { api } from 'boot/axios';

const columns = [
  {
    name: 'id',
    align: 'left',
    label: '#',
    field: 'id',
    sortable: true,
  },
  {
    name: 'account_id',
    align: 'left',
    label: '№ клиента',
    field: 'account_id',
  },
  {
    name: 'account_name',
    align: 'left',
    label: 'Название аккаунта',
    field: 'account_name',
  },
  {
    name: 'last_auth_date',
    align: 'left',
    label: 'Последняя авторизация',
    field: 'last_auth_date',
  },
  {
    name: 'last_error_date',
    align: 'left',
    label: 'Последняя ошибка',
    field: 'last_error_date',
  },
  {
    name: 'expires',
    align: 'left',
    label: 'Срок действия токена',
    field: 'expires',
  },
];

export default defineComponent({
  components: {},
  setup() {
    const tableRef = ref();
    const rows = ref([]);
    const filter = ref('');
    const loading = ref(true);
    const log_modules = ref([]);
    const log_module = ref(null);
    const clients = ref([]);
    const client = ref(null);
    const from = ref(null);
    const to = ref(null);

    const pagination = ref({
      sortBy: 'desc',
      descending: false,
      page: 1,
      rowsPerPage: 25,
    });

    function onRequest(props) {
      const { page, rowsPerPage, sortBy, descending } = props.pagination;
      console.log('onRequest called with:', {
        page,
        rowsPerPage,
        sortBy,
        descending,
      });
      console.log(props);

      // Установите значение page перед выполнением запроса
      pagination.value.page = page;
      pagination.value.rowsPerPage = rowsPerPage;

      loading.value = true;

      // Формируем базовый URL запроса
      let apiUrl =
        'https://ingeni.app/api/?clients&page=' +
        page +
        '&limit=' +
        rowsPerPage;

      // Добавляем параметры client и log_module, если они не пустые
      if (client.value) {
        apiUrl += '&client=' + client.value;
      }

      if (log_module.value) {
        apiUrl += '&module=' + log_module.value;
      }

      // Добавляем параметры client и log_module, если они не пустые
      if (from.value) {
        apiUrl += '&from=' + from.value;
      }

      if (to.value) {
        apiUrl += '&to=' + to.value;
      }

      api
        .get(apiUrl, {
          headers: { Authorization: 'Bearer Sceh~Bst##1DaKFR}fCv' },
        })
        .then((response) => {
          rows.value = response.data.records;
          pagination.value.rowsNumber = response.data.count;
          loading.value = false;
        })
        .catch((e) => {
          console.log(e);
          loading.value = false;
        });

      // Формируем базовый URL запроса
      let ApiUrlLog = 'https://ingeni.app/api/?log_modules';

      // Добавляем параметры client и log_module, если они не пустые
      if (client.value) {
        ApiUrlLog += '&client=' + client.value;
      }

      api
        .get(ApiUrlLog, {
          headers: { Authorization: 'Bearer Sceh~Bst##1DaKFR}fCv' },
        })
        .then((response) => {
          log_modules.value = response.data.records;
          // pagination.value.rowsNumber = response.data.count;
          // loading.value = false;
        })
        .catch((e) => {
          console.log(e);
          // loading.value = false;
        });

      api
        .get('https://ingeni.app/api/?clients', {
          headers: { Authorization: 'Bearer Sceh~Bst##1DaKFR}fCv' },
        })
        .then((response) => {
          clients.value = response.data.records;
          // pagination.value.rowsNumber = response.data.count;
          // loading.value = false;
        })
        .catch((e) => {
          console.log(e);
          // loading.value = false;
        });
    }

    onMounted(() => {
      tableRef.value.requestServerInteraction();
    });

    return {
      tableRef,
      pagination,
      loading,
      filter,
      rows,
      log_modules,
      log_module,
      clients,
      client,
      from,
      to,
      columns,
      name: 'PageIndex',
      onRequest,
    };
  },
});
</script>
<style scoped>
.fixed-table {
  table-layout: fixed; /* Фиксированный размер таблицы */
  width: 100%; /* Ширина таблицы 100% */
}

.fixed-td {
  width: 25%; /* Равномерное распределение на 4 колонки */
  white-space: nowrap; /* Запрет переноса текста */
  overflow: hidden; /* Обрезание текста, если он не помещается */
  text-overflow: ellipsis; /* Вывод многоточия, если текст обрезается */
}
</style>
