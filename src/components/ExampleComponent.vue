<template>
  <div class="full-width full-height q-pa-sm">
    <q-table
      class=""
      title="Poker Logs"
      :rows="data"
      :columns="columns"
      row-key="name"
    >
      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td key="name" :props="props">
            {{ props.row.name }}
          </q-td>
          <q-td key="value" :props="props">
            {{ props.row.value }}
          </q-td>
          <q-td key="action" :props="props">
            <div>
              <q-btn color="primary" outline label="Buy" class="q-ma-xs" />
            </div>

            <div>
              <q-btn outline color="red-6" label="Pay" class="q-ma-xs" />
            </div>
          </q-td>
        </q-tr>
      </template>
    </q-table>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue';
import { Todo, Meta } from './models';
import { useQuery, useResult } from '@vue/apollo-composable';
import gql from 'graphql-tag';
const { result, loading, error } = useQuery(gql`
  query GetTransaction {
    user {
      id
      name
      user_transactions {
        value
        type
      }
    }
  }
`);
const data = computed(() => result.value?.user);
console.log('result', result);

const clickCount = ref(0);
function increment() {
  clickCount.value += 1;
  return clickCount.value;
}
const columns: any[] = [
  {
    name: 'name',
    required: true,
    label: 'Player',
    align: 'left',
    field: 'name',
    sortable: true,
  },
  {
    name: 'value',
    align: 'center',
    label: 'Chip',
    field: 'value',
    sortable: true,
  },
  { name: 'action', label: 'Action', field: 'action' },
];
</script>
<style lang="scss" scoped></style>
