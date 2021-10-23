<script setup lang="ts">
const props = defineProps<{
  activePage: number
  count: number
  rowsPerPage: number
  totalPages: number
  setActivePage: Function
}>();

const beginning = computed(() => props.activePage === 1 ? 1 : props.rowsPerPage * (props.activePage - 1) + 1);
const end = computed(() => props.activePage === props.totalPages ? props.count : beginning.value + props.rowsPerPage - 1);

</script>

<template>
  <div>
    <div class="flex justify-between mt-10">
      <button :disabled="activePage === 1" @click="setActivePage(1)">
        ⏮️ First
      </button>
      <button :disabled="activePage === 1" @click="setActivePage(activePage - 1)">
        ⬅️ Previous
      </button>
      <button :disabled="activePage === totalPages" @click="setActivePage(activePage + 1)">
        Next ➡️
      </button>
      <button :disabled="activePage === totalPages" @click="setActivePage(totalPages)">
        Last ⏭️
      </button>
    </div>
    <div class="my-10">
      <p>
        Page {{ activePage }} of {{ totalPages }}
      </p>
      <p v-if="beginning === end">
        Rows: {{ end }}
      </p>
      <p v-else>
        Rows: {{ beginning }} - {{ end }} of {{ count }}
      </p>
    </div>
  </div>
</template>
