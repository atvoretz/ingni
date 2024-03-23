<template>
  <q-page class="">
    <q-table
      title="Входящие запросы"
      ref="tableRef"
      :rows="rows"
      :columns="columns"
      row-key="id"
      v-model:pagination="pagination"
      @request="onRequest"
      binary-state-sort
    >
      <template v-slot:top="props">
        <div class="q-gutter-md" style="width: 100%">
          <div class="row items-center">
            <div class="col-12 col-md-6 col-lg-3 q-pa-xs">
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
                @update:model-value="onRequest(props)"
                dense
              />
            </div>

            <div class="col-12 col-md-6 col-lg-3 q-pa-xs">
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
                @update:model-value="onRequest(props)"
                dense
              />
            </div>

            <!-- Дополнительные фильтры -->
            <div class="col-12 col-md-6 col-lg-3 q-pa-xs">
              <q-input
                outlined
                clearable
                v-model="from"
                dense
                @update:model-value="onRequest(props)"
              >
                <template v-slot:prepend>
                  <q-icon name="event" class="cursor-pointer">
                    <q-popup-proxy
                      cover
                      transition-show="scale"
                      transition-hide="scale"
                      dense
                    >
                      <q-date
                        v-model="from"
                        mask="YYYY-MM-DD HH:mm:ss"
                        @update:model-value="onRequest(props)"
                        dense
                      >
                        <div class="row items-center justify-end">
                          <q-btn
                            v-close-popup
                            label="Закрыть"
                            color="primary"
                            flat
                          />
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
                          <q-btn
                            v-close-popup
                            label="Закрыть"
                            color="primary"
                            flat
                          />
                        </div>
                      </q-time>
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>
            </div>

            <div class="col-12 col-md-6 col-lg-3 q-pa-xs">
              <q-input
                outlined
                clearable
                dense
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
                          <q-btn
                            v-close-popup
                            label="Закрыть"
                            color="primary"
                            flat
                          />
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
                          <q-btn
                            v-close-popup
                            label="Закрыть"
                            color="primary"
                            flat
                          />
                        </div>
                      </q-time>
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>
            </div>
            <div class="col-12 col-md-6 col-lg-3 q-pa-xs">
              <q-toggle
                @update:model-value="autoRefresh"
                label="Автообновление"
              ></q-toggle>
            </div>
          </div>
        </div>
      </template>
      <template v-slot:header="props">
        <q-tr :props="props" class="hide-on-mobile">
          <q-th
            v-for="col in props.cols"
            :key="col.name"
            :props="props"
            :class="col.colWidth"
          >
            <b>{{ col.label }}</b>
          </q-th>
        </q-tr>
      </template>
      <!-- Vue template -->
      <template v-slot:body="props">
        <q-tr
          v-if="$q.screen.gt.xs"
          :props="props"
          :class="{ 'highlighted-row': props.row._isNew }"
        >
          <q-td key="rowNumber" :props="props" class="fixed-td-number">
            {{
              props.rowIndex +
              1 +
              (pagination.page - 1) * pagination.rowsPerPage
            }}
          </q-td>
          <q-td key="date_time" :props="props" class="fixed-td">
            {{ props.row.date_time }}
            <q-badge color="info">{{ props.row.v_machine }}</q-badge>
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
            <q-icon
              size="sm"
              @click="props.row._showDetails = !props.row._showDetails"
              :name="
                props.row._showDetails
                  ? 'keyboard_arrow_up'
                  : 'keyboard_arrow_down'
              "
            ></q-icon>
          </q-td>
        </q-tr>
        <q-tr v-if="$q.screen.gt.xs" v-show="props.row._showDetails">
          <q-td colspan="1"></q-td>
          <q-td colspan="2" class="fixed-td-n" style="vertical-align: top">
            <JsonViewer :value="props.row.headers" copyable sort />
          </q-td>
          <q-td colspan="2" class="fixed-td-n" style="vertical-align: top">
            <JsonViewer :value="props.row.body" :expand-depth="4" copyable />
          </q-td>
        </q-tr>
        <!-- Адаптированный макет для экранов меньше или равных XS -->
        <q-tr v-if="!$q.screen.gt.xs" :props="props">
          <q-td class="q-pa-none" colspan="100%">
            <div class="q-pa-sm">
              <div>
                {{
                  props.rowIndex +
                  1 +
                  (pagination.page - 1) * pagination.rowsPerPage
                }}
              </div>

              <div>
                <strong>Дата и время:</strong> {{ props.row.date_time }}
              </div>
              <div><strong>Аккаунт:</strong> {{ props.row.account_name }}</div>
              <div><strong>Модуль:</strong> {{ props.row.module }}</div>
              <div>
                <strong>Код ответа:</strong>
                <span
                  :class="{
                    'red-text': props.row.code.charAt(0) !== '2',
                    'green-text': props.row.code.charAt(0) === '2',
                  }"
                >
                  {{ props.row.code }}
                </span>
                <q-icon
                  size="sm"
                  @click="props.row._showDetails = !props.row._showDetails"
                  :name="
                    props.row._showDetails
                      ? 'keyboard_arrow_up'
                      : 'keyboard_arrow_down'
                  "
                ></q-icon>
              </div>
              <div v-if="!$q.screen.gt.xs" v-show="props.row._showDetails">
                <div class="text-h6" style="margin-bottom: 10px">
                  Заголовок запроса
                </div>
                <!-- Заголовок для JsonViewer -->
                <JsonViewer :value="props.row.headers" copyable sort />
                <div class="text-h6" style="margin-bottom: 10px">
                  Тело запроса
                </div>
                <!-- Заголовок для JsonViewer -->
                <JsonViewer
                  :value="props.row.body"
                  :expand-depth="4"
                  copyable
                />
              </div>

              <!-- ... другая информация ... -->
            </div>
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
    <q-inner-loading
      :showing="loading"
      label="Спрашиваю в базе..."
      label-class="text-primary"
      label-style="font-size: 1.1em"
      color="primary"
      class="fixed-loading-container"
    />
  </q-page>
</template>

<script>
import JsonViewer from 'vue-json-viewer';
import { ref, onMounted, watch, onUnmounted } from 'vue';
import { defineComponent } from 'vue';
import { api } from 'boot/axios';

const columns = [
  {
    name: 'rowNumber',
    align: 'left',
    label: '#',
    field: 'rowNumber',
    sortable: true,
    colWidth: 'fixed-td-number',
  },
  {
    name: 'date_time',
    align: 'left',
    label: 'Период',
    field: 'period',
    sortable: true,
    colWidth: 'fixed-td',
  },
  {
    name: 'account_name',
    align: 'left',
    label: 'Название аккаунта',
    field: 'account_name',
    colWidth: 'fixed-td',
  },
  {
    name: 'module',
    align: 'left',
    label: 'Модуль',
    field: 'module',
    colWidth: 'fixed-td',
  },
  {
    name: 'code',
    align: 'left',
    label: 'Код ответа',
    field: 'code',
    colWidth: 'fixed-td',
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
    const previousRows = ref([]);
    const autoRefresh = ref(false);
    const autoRefreshInterval = ref(null);

    const pagination = ref({
      sortBy: 'desc',
      descending: false,
      page: 1,
      rowsPerPage: 25,
    });

    function onRequest(props) {
      loading.value = true; // Показываем спиннер перед запросами

      const { page, rowsPerPage, sortBy, descending } = props.pagination;
      console.log('onRequest called with:', {
        page,
        rowsPerPage,
        sortBy,
        descending,
      });

      pagination.value.page = page;
      pagination.value.rowsPerPage = rowsPerPage;

      // Формируем URL для основного запроса данных
      let apiUrl = `https://ingeni.app/api/?log&page=${page}&limit=${rowsPerPage}`;
      apiUrl += client.value ? `&client=${client.value}` : '';
      apiUrl += log_module.value ? `&module=${log_module.value}` : '';
      apiUrl += from.value ? `&from=${from.value}` : '';
      apiUrl += to.value ? `&to=${to.value}` : '';

      const mainRequest = api
        .get(apiUrl, {
          headers: { Authorization: 'Bearer Sceh~Bst##1DaKFR}fCv' },
        })
        .then((response) => {
          const newRows = response.data.records;

          // Сравниваем каждую новую строку со списком предыдущих строк
          newRows.forEach((newRow) => {
            const isNewRow = !previousRows.value.some(
              (previousRow) => previousRow.id === newRow.id
            );
            newRow._isNew = isNewRow; // Прямое присваивание для Vue 3
          });

          // Обновляем текущий набор строк интерфейса
          rows.value = newRows;

          // Сохраняем копию новых строк как предыдущие для следующего сравнения
          previousRows.value = newRows.map((row) => ({ ...row }));

          pagination.value.rowsNumber = response.data.count;
        });

      let logModulesApiUrl = 'https://ingeni.app/api/?log_modules';

      // Добавляем параметры к URL в зависимости от наличия значений в переменных
      if (client.value) {
        logModulesApiUrl += `&client=${client.value}`;
      }
      if (from.value) {
        logModulesApiUrl += `&from=${from.value}`;
      }
      if (to.value) {
        logModulesApiUrl += `&to=${to.value}`;
      }

      const logModulesRequest = api
        .get(logModulesApiUrl, {
          headers: { Authorization: 'Bearer Sceh~Bst##1DaKFR}fCv' },
        })
        .then((response) => {
          log_modules.value = response.data.records;
        });

      const clientsRequest = api
        .get('https://ingeni.app/api/?clients', {
          headers: { Authorization: 'Bearer Sceh~Bst##1DaKFR}fCv' },
        })
        .then((response) => {
          clients.value = response.data.records;
        });

      // Дожидаемся завершения всех запросов
      Promise.all([mainRequest, logModulesRequest, clientsRequest])
        .catch((error) => {
          console.error(error);
        })
        .finally(() => {
          loading.value = false; // Скрываем спиннер после завершения всех запросов
        });
    }

    onMounted(() => {
      tableRef.value.requestServerInteraction();
    });

    watch(autoRefresh, (newValue) => {
      // Очистка существующего интервала при любом изменении autoRefresh
      if (autoRefreshInterval.value) {
        clearInterval(autoRefreshInterval.value);
        autoRefreshInterval.value = null; // Если используется ref
      }

      if (newValue) {
        onRequest(); // Немедленно выполнить запрос
        autoRefreshInterval.value = setInterval(() => {
          onRequest(); // Повторный запрос каждые X миллисекунд
        }, 10000); // Например, каждые 10 секунд
      }
    });

    onUnmounted(() => {
      if (autoRefreshInterval.value) clearInterval(autoRefreshInterval.value);
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
      autoRefresh,
      autoRefreshInterval,
    };
  },
});
</script>

<style scoped>
@media (max-width: 767px) {
  .q-table .q-table__container .q-table__middle {
    overflow-x: auto;
  }
}

@media (max-width: 767px) {
  .hide-on-mobile {
    display: none;
  }
}

.fixed-td-number {
  width: 4%; /* Равномерное распределение на 4 колонки */
  white-space: nowrap; /* Запрет переноса текста */
  overflow: hidden; /* Обрезание текста, если он не помещается */
  text-overflow: ellipsis; /* Вывод многоточия, если текст обрезается */
}

.fixed-td {
  width: 24%; /* Равномерное распределение на 4 колонки */
  white-space: nowrap; /* Запрет переноса текста */
  overflow: hidden; /* Обрезание текста, если он не помещается */
  text-overflow: ellipsis; /* Вывод многоточия, если текст обрезается */
}

.fixed-td-n {
  width: 48%; /* Равномерное распределение на 4 колонки */
  white-space: nowrap; /* Запрет переноса текста */
  overflow: hidden; /* Обрезание текста, если он не помещается */
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

.fixed-loading-container {
  position: fixed; /* Фиксированное позиционирование относительно окна браузера */
  top: 0;
  left: 0;
  width: 100vw; /* Ширина равна 100% от ширины вьюпорта */
  height: 100vh; /* Высота равна 100% от высоты вьюпорта */
  display: flex;
  justify-content: center; /* Горизонтальное центрирование содержимого */
  align-items: center; /* Вертикальное центрирование содержимого */
  z-index: 10000; /* Убедитесь, что контейнер находится поверх других элементов */
}

.highlighted-row {
  animation: highlight-row 5s forwards;
}

@keyframes highlight-row {
  0% {
    background-color: yellow;
  }
  100% {
    background-color: transparent;
  }
}
</style>
