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
            Input value:
            <br />
            {{ props.row.value ?? 'N/A' }}
            <br />
            Output value:
            <br />
            {{ props.row?.finalValue ?? 'N/A' }}
            <q-popup-edit
              v-model.number="props.row.finalValue"
              v-slot="scope"
              buttons
              @save="(value) => submitFinalResult(props.row.id, value)"
            >
              <q-input
                v-model.number="scope.value"
                dense
                autofocus
                type="number"
                @keyup.enter="scope.set"
              />
            </q-popup-edit>
          </q-td>
          <q-td key="action" :props="props">
            <div>
              <q-btn
                @click="
                  insertTransaction({
                    value: -100,
                    type: 'BUY',
                    userId: props.row?.id,
                  })
                "
                color="primary"
                outline
                label="Buy"
                class="q-ma-xs"
              />
            </div>

            <div>
              <q-btn
                @click="
                  insertTransaction({
                    value: 100,
                    type: 'PAY',
                    userId: props.row?.id,
                  })
                "
                outline
                color="red-6"
                label="Pay"
                class="q-ma-xs"
              />
            </div>
          </q-td>
        </q-tr>
      </template>
    </q-table>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { useQuery, useMutation } from '@vue/apollo-composable';
import { format, formatISO } from 'date-fns';

import gql from 'graphql-tag';
const GET_USER_DATA = gql`
  query GetUser {
    user {
      id
      name
      user_transactions(where: { type: { _eq: "FINAL" } }) {
        value
      }
      user_transactions_aggregate(where: { type: { _neq: "FINAL" } }) {
        aggregate {
          sum {
            value
          }
        }
      }
    }
  }
`;

const INSERT_TRANSACTION = gql`
  mutation InsertTransaction(
    $time: date
    $userId: Int
    $type: String
    $timeStamp: timestamptz
    $value: Int
  ) {
    insert_transaction(
      objects: {
        time: $time
        userId: $userId
        type: $type
        timeStamp: $timeStamp
        value: $value
      }
    ) {
      affected_rows
      returning {
        time
        id
        userId
        value
        type
        timeStamp
      }
    }
  }
`;

const { result, loading, error, refetch } = useQuery(GET_USER_DATA);
const { mutate: insertTransactionMutation } = useMutation(INSERT_TRANSACTION);

const data = computed(() =>
  result.value?.user.map(
    ({ id, name, user_transactions_aggregate, user_transactions }: any) => ({
      id,
      name,
      value: user_transactions_aggregate?.aggregate?.sum?.value,
      finalValue: user_transactions?.[0]?.value,
    })
  )
);

const insertTransaction = async ({ value, userId, type }: any) => {
  const transaction = {
    time: format(new Date(), 'yyyy-MM-dd'),
    timeStamp: formatISO(new Date()),
    userId,
    type,
    value,
  };

  await insertTransactionMutation(transaction);
  refetch();
};

const submitFinalResult = async (userId: number, value: number) => {
  const transaction = {
    time: format(new Date(), 'yyyy-MM-dd'),
    timeStamp: formatISO(new Date()),
    userId,
    type: 'FINAL',
    value,
  };

  await insertTransactionMutation(transaction);
  refetch();
};

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
