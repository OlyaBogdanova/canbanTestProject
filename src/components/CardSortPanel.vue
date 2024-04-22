<template>
  <div class="d-flex align-center mb-2">
    <div class="sort__title">Рейтинг</div>

    <v-tooltip v-model="showTooltip" location="top center" origin="auto">
      <template v-slot:activator="{ props }">
        <v-btn
          icon
          v-bind="props"
          variant="plain"
          rounded="sm"
          @click="changeSortState"
        >
          <v-icon color="white">
            {{ sortActiveState.icon }}
          </v-icon>
        </v-btn>
      </template>
      <span> {{ sortActiveState.title }}</span>
    </v-tooltip>
  </div>
</template>
<script setup>
import { ref, computed } from "vue";

const emit = defineEmits(["sort"]);
const sortStates = [
  { title: "Без сортировки", icon: "mdi-sort", id: "none" },
  { title: "По убыванию рейтинга", icon: "mdi-sort-ascending", id: "desc" },
  { title: "По возрастанию рейтинга", icon: "mdi-sort-descending", id: "asc" },
];
const showTooltip = ref(false);
const sortActiveState = ref(sortStates[0]);
const computedActiveStateIdx = computed(() =>
  sortStates.findIndex((state) => state.id === sortActiveState.value.id)
);

function changeSortState() {
  if (computedActiveStateIdx.value < sortStates.length - 1) {
    sortActiveState.value = sortStates[computedActiveStateIdx.value + 1];
  } else {
    sortActiveState.value = sortStates[0];
  }
  emit("sort", sortActiveState.value.id);
}
</script>

<style scoped>
.sort__title {
  color: white;
  margin-right: 5px;
}
</style>
