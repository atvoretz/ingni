<template>
  <q-page class="">
    <div class="q-pa-lg flex flex-center">
      <q-pagination v-model="page" :max="5" />
    </div>
    <q-table
      title="Входящие запросы"
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
        <q-tr v-show="props.row._showDetails">
          <q-td colspan="50%" style="vertical-align: top">
            <span
              v-for="(value, name) in props.row.headers"
              :key="value.valueId"
            >
              <b>{{ name }}</b> : {{ value }} <br />
            </span>
          </q-td>
          <q-td colspan="50%" style="vertical-align: top">
            <pre>{{ props.row.body }}</pre>
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
    <div class="q-pa-lg flex flex-center">
      <q-pagination v-model="current" :max="5" />
    </div>
  </q-page>
</template>

<script>
import { ref } from 'vue';
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
    sortable: true,
  },
  {
    name: 'module',
    align: 'left',
    label: 'Модуль',
    field: 'module',
    sortable: true,
  },
  {
    name: 'code',
    align: 'left',
    label: 'Код ответа',
    field: 'code',
    sortable: true,
  },
];

export default defineComponent({
  setup() {
    return {
      pagination: {
        sortBy: 'desc',
        descending: false,
        page: 1,
        rowsPerPage: 25,
        rowsNumber: 100,
      },
    };
  },
  name: 'PageIndex',
  data() {
    return {
      rows: [],
      columns,
    };
  },
  created() {
    api
      .get('https://ingeni.app/api/?log', {
        headers: { Authorization: 'Bearer Sceh~Bst##1DaKFR}fCv' },
      })
      .then((response) => {
        this.rows = response.data.records;
        this.initialPagination.rowsNumber = response.data.count;
      })
      .catch(() => {});
  },
});
</script>
