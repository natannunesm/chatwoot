<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import { useAccount } from 'dashboard/composables/useAccount';

const { accountId } = useAccount();

const conversations = ref([]);
const loading = ref(true);

// Colunas do Kanban baseadas em tags
const columns = [
  { id: 'novo', title: 'Novo' },
  { id: 'andamento', title: 'Em atendimento' },
  { id: 'aguardando', title: 'Aguardando cliente' },
  { id: 'followup', title: 'Follow-up' },
  { id: 'finalizado', title: 'Finalizado' },
];

// Puxa todas as conversas da API
const fetchConversations = async () => {
  loading.value = true;

  const res = await axios.get(
    `/api/v1/accounts/${accountId}/conversations`,
  );

  conversations.value = res.data.payload;
  loading.value = false;
};

// Filtra conversas por tag
const filteredByTag = tag =>
  computed(() =>
    conversations.value.filter(c =>
      c.labels?.includes(tag)
    )
  );

// Atualiza tag ao mover cartão
const updateConversationTags = async (conversationId, newTag) => {
  await axios.put(
    `/api/v1/accounts/${accountId}/conversations/${conversationId}`,
    {
      labels: [newTag],
    }
  );
  fetchConversations();
};

onMounted(fetchConversations);
</script>

<template>
  <div class="p-6">

    <h1 class="text-2xl font-bold mb-4">CRM Kanban</h1>

    <div v-if="loading" class="text-gray-500">Carregando conversas...</div>

    <div class="grid grid-cols-5 gap-4" v-else>
      <div
        v-for="col in columns"
        :key="col.id"
        class="bg-gray-100 rounded-lg p-3 h-[80vh] overflow-y-auto shadow"
      >
        <h2 class="text-lg font-semibold mb-3">{{ col.title }}</h2>

        <!-- Listagem dos cards -->
        <div
          v-for="c in filteredByTag(col.id)"
          :key="c.id"
          class="bg-white p-3 rounded shadow mb-3 cursor-pointer border border-gray-200"
        >
          <p class="text-sm font-semibold">#{{ c.id }}</p>
          <p class="text-xs">{{ c.meta.sender?.name }}</p>

          <select
            class="mt-2 w-full p-1 border rounded"
            @change="updateConversationTags(c.id, $event.target.value)"
          >
            <option
              v-for="col2 in columns"
              :key="col2.id"
              :value="col2.id"
              :selected="col.id === col2.id"
            >
              Mover para: {{ col2.title }}
            </option>
          </select>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
/* Apenas para deixar mais limpo */
::-webkit-scrollbar {
  width: 6px;
}
::-webkit-scrollbar-thumb {
  background: #cccccc;
  border-radius: 10px;
}
</style>
