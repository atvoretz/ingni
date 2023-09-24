<template>
  <q-page class="">
    <q-table
      title="Пойманные сбои"
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
      <template v-slot:top="props">
        <q-select
          outlined
          clearable
          v-model="client"
          :options="clients"
          option-value="account_id"
          option-label="clients_for_select"
          label="Клиенты"
          emit-value
          map-options
          style="min-width: 250px"
          @update:model-value="onRequest(props)"
        />
        <q-space />
        <q-select
          outlined
          clearable
          v-model="log_module"
          :options="log_modules"
          label="Модули"
          option-value="module"
          option-label="module"
          emit-value
          map-options
          style="min-width: 250px"
          @update:model-value="onRequest(props)"
        />
        <q-space />
        <q-input
          outlined
          clearable
          v-model="from"
          @update:model-value="onRequest(props)"
        >
          <template v-slot:prepend>
            <q-icon name="event" class="cursor-pointer">
              <q-popup-proxy
                cover
                transition-show="scale"
                transition-hide="scale"
              >
                <q-date
                  v-model="from"
                  mask="YYYY-MM-DD HH:mm:ss"
                  @update:model-value="onRequest(props)"
                >
                  <div class="row items-center justify-end">
                    <q-btn v-close-popup label="Close" color="primary" flat />
                  </div>
                </q-date>
              </q-popup-proxy>
            </q-icon>
          </template>
          <q-space />
          <template v-slot:append>
            <q-icon name="access_time" class="cursor-pointer">
              <q-popup-proxy
                cover
                transition-show="scale"
                transition-hide="scale"
              >
                <q-time
                  v-model="from"
                  mask="YYYY-MM-DD HH:mm:ss"
                  format24h
                  @update:model-value="onRequest(props)"
                >
                  <div class="row items-center justify-end">
                    <q-btn v-close-popup label="Закрыть" color="primary" flat />
                  </div>
                </q-time>
              </q-popup-proxy>
            </q-icon>
          </template>
        </q-input>
        <q-space />
        <q-input
          outlined
          clearable
          v-model="to"
          @update:model-value="onRequest(props)"
        >
          <template v-slot:prepend>
            <q-icon name="event" class="cursor-pointer">
              <q-popup-proxy
                cover
                transition-show="scale"
                transition-hide="scale"
              >
                <q-date
                  v-model="to"
                  mask="YYYY-MM-DD HH:mm:ss"
                  @update:model-value="onRequest(props)"
                >
                  <div class="row items-center justify-end">
                    <q-btn v-close-popup label="Close" color="primary" flat />
                  </div>
                </q-date>
              </q-popup-proxy>
            </q-icon>
          </template>

          <template v-slot:append>
            <q-icon name="access_time" class="cursor-pointer">
              <q-popup-proxy
                cover
                transition-show="scale"
                transition-hide="scale"
              >
                <q-time
                  v-model="to"
                  mask="YYYY-MM-DD HH:mm:ss"
                  format24h
                  @update:model-value="onRequest(props)"
                >
                  <div class="row items-center justify-end">
                    <q-btn v-close-popup label="Close" color="primary" flat />
                  </div>
                </q-time>
              </q-popup-proxy>
            </q-icon>
          </template>
        </q-input>
        <q-space />
        <!-- <q-input
          borderless
          dense
          debounce="300"
          color="primary"
          v-model="filter"
        >
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input> -->
      </template>

      <template v-slot:header="props">
        <q-tr :props="props">
          <q-th
            v-for="col in props.cols"
            :key="col.name"
            :props="props"
            class="fixed-td"
          >
            <b>{{ col.label }}</b>
          </q-th>
        </q-tr>
      </template>
      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td key="date_time" :props="props" class="fixed-td">
            {{ props.row.date_time }}
          </q-td>
          <q-td key="account_name" :props="props" class="fixed-td">
            {{ props.row.account_name }}
          </q-td>
          <q-td key="module" :props="props" class="fixed-td">
            {{ props.row.module }}
          </q-td>
          <q-td key="code" :props="props" class="fixed-td">
            <span
              :class="{
                'red-text': props.row.code.charAt(0) !== '2',
                'green-text': props.row.code.charAt(0) === '2',
              }"
            >
              {{ props.row.code }}
            </span>
          </q-td>
          <q-td key="message" :props="props" class="fixed-td">
            {{ props.row.message }}
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
  {
    name: 'message',
    align: 'left',
    label: 'Ответ',
    field: 'message',
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
        'https://ingeni.app/api/?errors&page=' + page + '&limit=' + rowsPerPage;

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
  width: 20%; /* Равномерное распределение на 4 колонки */
  overflow: hidden; /* Обрезание текста, если он не помещается */
  word-wrap: break-word;
  text-overflow: ellipsis; /* Вывод многоточия, если текст обрезается */
}

/* Добавьте стили для красного текста и жирного шрифта */
.red-text {
  color: red;
  font-weight: bold; /* Добавление жирного шрифта */
}

.green-text {
  color: green;
  font-weight: bold;
}
</style>
