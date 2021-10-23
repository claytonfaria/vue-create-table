<template>
  <div>
    <table>
      <thead>
        <tr>
          <th v-for="column in props.columns" :key="column.accessor">
            <span> {{ column.label }}</span>
            <button @click="handleSort(column.accessor )">
              {{ getSortIcon(column.accessor,sort) }}
            </button>
          </th>
        </tr>
        <tr>
          <th v-for="column in props.columns" :key="column.accessor">
            <input v-model="filters[column.accessor]" type="search" :placeholder="column.label" class="text-black px-4">
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in calculatedRows" :key="row.id">
          <td v-for="column in props.columns" :key="column.accessor">
            <template v-if="column.format">
              {{ column.format(row[column.accessor]) }}
            </template>
            <template v-else>
              {{ row[column.accessor] }}
            </template>
          </td>
        </tr>
      </tbody>
    </table>
    <Pagination :active-page="activePage" :count="count" :rows-per-page="rowsPerPage" :total-pages="totalPages" :set-active-page="setActivePage"></Pagination>
  </div>
</template>

<script setup lang="ts">

import { isBoolean, isEmpty, isNumber, isString, orderBy, toLower } from 'lodash';

export interface Column {
  accessor: 'id' | 'name' | 'age' | 'is_manager' | 'start_date'
  label: string
  format?: (value: any) => '✔️' | '✖️'
}

interface Row {
  id: number
  name: string | null
  age: number | null
  is_manager: boolean | null
  start_date: string | null
}

interface TableProps {
  rows: Row[]
  columns: Column[]
}
const props = defineProps<TableProps>();

const filters = reactive({} as Record<Partial<Column['accessor']>, string>);
const activePage = ref(1);

const sort: {order: 'asc' | 'desc'; orderBy: string} = reactive({ order: 'asc', orderBy: 'id' });

const filteredRows = computed(() => filterRows(props.rows, filters));
const sortedRows = computed(() => sortRows(filteredRows.value, sort));

const rowsPerPage = ref(3);
const count = computed(() => filteredRows.value.length);
const totalPages = computed(() => Math.ceil(count.value / rowsPerPage.value));
const calculatedRows = computed(() => sortedRows.value.slice((activePage.value - 1) * rowsPerPage.value, activePage.value * rowsPerPage.value));

const setActivePage = (page: number) => {
  activePage.value = page;
};

watch(filters, (newValue, oldValue) => {
  if (activePage.value !== 1) {
    setActivePage(1);
  }
});

function sortRows(rows: Row[], sort: { order: 'asc' | 'desc'; orderBy: string }) {
  return orderBy(rows, sort.orderBy, sort.order);
}

function handleSort(accessor: Column['accessor']) {
  setActivePage(1);

  sort.order = sort.order === 'asc' && sort.orderBy === accessor ? 'desc' : 'asc';
  sort.orderBy = accessor;
}

function getSortIcon(accessor: Column['accessor'], sort: { order: 'asc' | 'desc'; orderBy: string }) {
  if (accessor === sort.orderBy) {
    if (sort.order === 'asc') {
      return '⬆️';
    }
    return '⬇️';
  }
  else {
    return '️↕️';
  }
}

function filterRows(rows: Row[], filters: Record<Partial<Column['accessor']>, string>) {
  if (isEmpty(filters)) return rows;
  const filtersWithoutEmptyKeys = Object.fromEntries(Object.entries(filters).filter(([_, v]) => Boolean(v)));

  return rows.filter((row) => {
    return Object.keys(filtersWithoutEmptyKeys).every((accessor) => {
      const value = row[accessor as Column['accessor']];
      const searchValue = filtersWithoutEmptyKeys[accessor as Column['accessor']];

      if (isString(value))
        return toLower(value).includes(toLower(searchValue));

      if (isBoolean(value))
        return (searchValue === 'true' && value) || (searchValue === 'false' && !value);

      if (isNumber(value))
        return String(value).includes(searchValue);

      return false;
    });
  });
}

</script>
