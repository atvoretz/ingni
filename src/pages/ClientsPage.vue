<template>
  <q-page class="">
    <q-table
      title="Входящие запросы"
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
          <q-th
            v-for="col in props.cols"
            :key="col.name"
            :props="props"
            colspan="25%"
          >
            <b>{{ col.label }}</b>
          </q-th>
        </q-tr>
      </template>
      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td key="date_time" :props="props" colspan="25%">
            {{ props.row.date_time }}
            <q-badge color="info">{{ props.row.v_machine }}</q-badge>
          </q-td>
          <q-td key="account_name" :props="props" colspan="25%">
            {{ props.row.account_name }}
          </q-td>
          <q-td key="module" :props="props" colspan="25%">
            {{ props.row.module }}
          </q-td>
          <q-td key="code" :props="props" colspan="25%">
            {{ props.row.code }}
          </q-td>
        </q-tr>
      </template>
      <template v-slot:top-right>
        <q-input
          borderless
          dense
          debounce="300"
          v-model="filter"
          placeholder="Search"
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
    name: 'date_time',
    align: 'left',
    label: 'Период',
    field: 'period',
    sortable: true,
  },
  {
    name: 'account_name',
    align: 'left',
    label: 'Название аккаунта',
    field: 'account_name',
  },
  {
    name: 'module',
    align: 'left',
    label: 'Модуль',
    field: 'module',
  },
  {
    name: 'code',
    align: 'left',
    label: 'Код ответа',
    field: 'code',
  },
];

export default defineComponent({
  components: {
    JsonViewer,
  },
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
        'https://ingeni.app/api/?log&page=' + page + '&limit=' + rowsPerPage;

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
