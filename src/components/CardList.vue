<template>
  <div class="d-flex flex-column">
    <CardSortPanel @sort="sortData" />
    <section
      :style="`background: ${options.color}`"
      @drop="onDrop($event, options.id)"
      @dragover.prevent
      @dragenter.prevent
    >
      <div class="title">
        <h2>
          {{ options.title }}
        </h2>
        <div class="counter">
          <span>{{ cards.length }}</span>
        </div>
      </div>
      <v-btn
        icon="mdi-plus"
        variant="tonal"
        class="mt-5"
        color="white"
        @click="isNewCardDialogOpen = true"
      />

      <CardItem
        v-for="(card, index) in cards"
        draggable="true"
        :key="index"
        :card="card"
        :options="props.options"
        @delete-card="deleteCard(card.id)"
        @dragstart="onDragStart($event, card)"
      />

      <CardForm
        title="Добавление новой карточки"
        v-model="isNewCardDialogOpen"
        :form="form"
        @save-card="addCard"
        @close-form="isNewCardDialogOpen = false"
      />
    </section>
  </div>
</template>

<script setup>
import { ref, inject, computed, watch } from "vue";
import CardItem from "./CardItem.vue";
import CardForm from "./CardForm.vue";
import CardSortPanel from "./CardSortPanel.vue";

const boardData = inject("boardData");
const props = defineProps({
  options: {},
});

const isNewCardDialogOpen = ref(false);

const form = ref({
  image: "",
  id: null,
  title: "",
  description: "",
  category: "",
  price: null,
  rating: {},
});

const initialCards = computed(() => {
  let cards;
  switch (props.options.id) {
    case 1:
      cards = boardData.value.firstList;
      break;
    case 2:
      cards = boardData.value.secondList;
      break;
    case 3:
      cards = boardData.value.lastList;
      break;
  }
  return cards;
});
const cards = ref([]);
watch(
  () => initialCards.value,
  () => {
    console.log("watch");
    cards.value = initialCards.value.map((a) => Object.assign({}, a));
  }
);

// function getLocalCards() {

//   console.log(cards.value)
// }
// getLocalCards();

function addCard() {
  cards.value.unshift(form.value);
  isNewCardDialogOpen.value = false;
}
function deleteCard(id) {
  cards.value = cards.value.filter((card) => card.id != id);
}
function onDragStart(event, item) {
  event.dataTransfer.dropEffect = "move";
  event.dataTransfer.effectAllowed = "move";
  event.dataTransfer.setData("itemId", item.id.toString());
}
function onDrop(event, optionsId) {
  console.log("onDrop", event, optionsId);
  const itemId = parseInt(event.dataTransfer.getData("itemId"));
  let otherLists = [];

  switch (optionsId) {
    case 1:
      cards.value = boardData.value.firstList.map((a) => Object.assign({}, a));
      otherLists = boardData.value.secondList.concat(boardData.value.lastList);
      break;
    case 2:
      cards.value = boardData.value.secondList.map((a) => Object.assign({}, a));
      otherLists = boardData.value.firstList.map((a) => Object.assign({}, a)).concat(boardData.value.lastList);
      break;
    case 3:
      cards.value = boardData.value.lastList.map((a) => Object.assign({}, a));
      otherLists = boardData.value.secondList.map((a) => Object.assign({}, a)).concat(boardData.value.firstList);
      break;
  }

  const newCard = otherLists.find((card) => card.id === itemId);

  boardData.value.firstList = boardData.value.firstList.filter(
    (card) => card.id !== newCard.id
  );
  boardData.value.secondList = boardData.value.secondList.filter(
    (card) => card.id !== newCard.id
  );
  boardData.value.lastList = boardData.value.lastList.filter(
    (card) => card.id !== newCard.id
  );
  cards.value.unshift(newCard);

  switch (optionsId) {
    case 1:
      boardData.value.firstList = cards.value;
      break;
    case 2:
      boardData.value.secondList = cards.value;
      break;
    case 3:
      boardData.value.lastList = cards.value;
      break;
  }
}

function sortData(sortValue) {
  switch (sortValue) {
    case "none":
      cards.value = initialCards.value.map((a) => Object.assign({}, a)) ;
      break;
    case "asc":
      cards.value = cards.value.sort((a, b) => a.rating.rate - b.rating.rate);
      break;
    case "desc":
      cards.value = cards.value.sort((a, b) => b.rating.rate - a.rating.rate);
      break;
  }
}
</script>

<style lang="scss" scoped>
section {
  padding: 10px;
  width: 400px;
  min-height: 500px;
  height: fit-content;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  .title {
    padding-bottom: 10px;
    width: 90%;
    display: flex;
    align-items: center;
    justify-content: center;
    border-bottom: 2px solid white;
    .counter {
      margin-left: 10px;
      background: white;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      display: flex;
      justify-content: center;
      align-items: center;
    }
  }
}
</style>
