<script setup>
import { ref } from "vue";
import * as solidIcons from "@fortawesome/free-solid-svg-icons";
import * as regularIcons from "@fortawesome/free-regular-svg-icons";
import * as brandsIcons from "@fortawesome/free-brands-svg-icons";

const configLevel = [
  {
    level: 1,
    gridClass: "grid grid-cols-4 gap-3",
    time: 60,
    sizeMatrix: 4,
    cardSizeClass: "w-24 h-24",
    iconSize: "lg",
  },
  {
    level: 2,
    gridClass: "grid grid-cols-4 gap-3",
    time: 30,
    sizeMatrix: 4,
    cardSizeClass: "w-24 h-24",
    iconSize: "lg",
  },
  {
    level: 3,
    gridClass: "grid grid-cols-6 gap-2",
    time: 60,
    sizeMatrix: 6,
    cardSizeClass: "w-16 h-16",
    iconSize: "sm",
  },
  {
    level: 4,
    gridClass: "grid grid-cols-6 gap-2",
    time: 40,
    sizeMatrix: 6,
    cardSizeClass: "w-16 h-16",
    iconSize: "sm",
  },
  {
    level: 5,
    gridClass: "grid grid-cols-8 gap-2",
    time: 60,
    sizeMatrix: 8,
    cardSizeClass: "w-14 h-14",
    iconSize: "2xs",
  },
  {
    level: 6,
    gridClass: "grid grid-cols-8 gap-2",
    time: 40,
    sizeMatrix: 8,
    cardSizeClass: "w-14 h-14",
    iconSize: "2xs",
  },
];

const activeLevel = ref(0);
const items = [];
const colorArray = ["#4a00ff", "#ff00d3", "#00d7c0", "#00b5ff", "#00a96e", "#ffbe00", "#ff5861"];
const resultTime = ref("");
const movesCount = ref(0);
const matchCount = ref(0);
const cardValues = ref([]);
const firstCard = ref();
const firstCardName = ref();
const secondCard = ref();
const secondCardName = ref();
const alertText = ref("");
const isStartGame = ref(false);

function getRandomIcon() {
  const iconArray = Object.values({ ...solidIcons, ...regularIcons, ...brandsIcons });
  const randomIndex = Math.floor(Math.random() * iconArray.length);
  return iconArray[randomIndex];
}

function timeGeneratorCustom(sec = 90) {
  sec = Math.max(0, Math.floor(sec));
  const minutes = Math.floor((sec % 3600) / 60)
    .toString()
    .padStart(2, "0");
  const seconds = (sec % 60).toString().padStart(2, "0");

  resultTime.value = `${minutes}:${seconds}`;

  const intervalId = setInterval(() => {
    if (sec > 0) {
      sec--;

      const minutes = Math.floor((sec % 3600) / 60)
        .toString()
        .padStart(2, "0");
      const seconds = (sec % 60).toString().padStart(2, "0");

      resultTime.value = `${minutes}:${seconds}`;
    } else {
      clearInterval(intervalId);
      alertText.value = "Time up, Game Over!";
      stopGame();
    }
  }, 1000);
}

//For calculating moves
function movesCounter() {
  movesCount.value += 1;
}

function getRandomColor(colors) {
  // Get a random index from the array
  const randomIndex = Math.floor(Math.random() * colors.length);

  // Return the randomly selected color
  return colors[randomIndex];
}

const generateRandom = (size = 4) => {
  for (let index = 0; index < size * size; index++) {
    const randomColor = getRandomColor(colorArray);
    const randomIcon = getRandomIcon();
    items.push({ name: randomIcon.iconName, icon: randomIcon, color: randomColor });
  }
  //temporary array
  let tempArray = [...items];
  //initializes cardValues array
  let tempCardValues = [];
  //size should be double (4*4 matrix)/2 since pairs of objects would exist
  size = (size * size) / 2;
  //Random object selection
  for (let i = 0; i < size; i++) {
    const randomIndex = Math.floor(Math.random() * tempArray.length);
    tempCardValues.push(tempArray[randomIndex]);
    //once selected remove the object from temp array
    tempArray.splice(randomIndex, 1);
  }
  return tempCardValues;
};

function handleClick(e, cardName) {
  const cardContainer = e.currentTarget.parentNode;
  if (!cardContainer.classList.contains("matched")) {
    cardContainer.classList.add("flipped");
    if (!firstCard.value) {
      firstCard.value = cardContainer;
      firstCardName.value = cardName;
    } else {
      movesCounter();

      secondCard.value = cardContainer;
      secondCardName.value = cardName;

      if (firstCardName.value == secondCardName.value) {
        firstCard.value.classList.add("matched");
        secondCard.value.classList.add("matched");

        matchCount.value += 1;

        if (matchCount.value == Math.floor(cardValues.value.length / 2)) {
          alertText.value = "You Won";
          stopGame();
        }
      } else {
        let [tempFirst, tempSecond] = [firstCard.value, secondCard.value];
        setTimeout(() => {
          tempFirst.classList.remove("flipped");
          tempSecond.classList.remove("flipped");
        }, 550);
      }
      firstCard.value = false;
      secondCard.value = false;
    }
  }
}

function startGame() {
  const randomCard = generateRandom(configLevel[activeLevel.value].sizeMatrix);
  cardValues.value = [...randomCard, ...randomCard];
  timeGeneratorCustom(configLevel[activeLevel.value].time);
  isStartGame.value = true;
}

function stopGame() {
  resultTime.value = "";
  cardValues.value = [];
  movesCount.value = 0;
  setTimeout(function () {
    alertText.value = "";
    isStartGame.value = false;
  }, 1000);
}
</script>

<template>
  <div class="w-screen h-screen flex justify-center items-center">
    <div class="card bg-white shadow-xl rounded-xl">
      <div v-if="isStartGame && !alertText" class="card-body">
        <div class="stats-container flex flex-col justify-center items-center mb-[1.2rem] font-semibold text-black">
          <div id="moves-count">Moves {{ movesCount }}</div>
          <div id="time" class="text-xl">Time {{ resultTime }}</div>
        </div>
        <div :class="configLevel[activeLevel].gridClass" id="game-container">
          <div v-for="card in cardValues" class="card-container cursor-pointer card" :class="configLevel[activeLevel].cardSizeClass">
            <div @click.self="(e) => handleClick(e, card.name)" class="card-before bg-warning rounded-xl border-4 border-black shadow-xl flex justify-center card-title text-[3rem] w-full h-full absolute backface-visibility-hidden text-black rotate-y-0">?</div>
            <div class="card-after bg-white rounded-xl border-4 border-black shadow-xl flex justify-center card-title text-[3rem] w-full h-full absolute backface-visibility-hidden text-black rotate-y-180">
              <font-awesome-icon :icon="[card.icon.prefix, card.icon.iconName]" :size="configLevel[activeLevel].iconSize" :style="{ color: card.color }" />
            </div>
          </div>
        </div>
      </div>
      <div v-if="alertText" id="alert" class="card-body flex justify-center items-center">
        <div class="card-title text-black font-semibold text-2xl">{{ alertText }}</div>
      </div>
      <div v-if="!isStartGame" id="controls-container" class="card-body flex flex-col justify-center items-center space-y-2">
        <div class="card-title text-black font-semibold text-xl">Welcome to Memory Game</div>
        <button class="btn btn-warning text-md font-semibold" @click="startGame">Start Game</button>
        <div class="flex gap-2">
          <button :class="activeLevel == 0 ? '' : 'btn-outline'" class="btn btn-square btn-xs btn-warning text-md font-semibold" @click="activeLevel = 0">1</button>
          <button :class="activeLevel == 1 ? '' : 'btn-outline'" class="btn btn-square btn-xs btn-warning text-md font-semibold" @click="activeLevel = 1">2</button>
          <button :class="activeLevel == 2 ? '' : 'btn-outline'" class="btn btn-square btn-xs btn-warning text-md font-semibold" @click="activeLevel = 2">3</button>
          <button :class="activeLevel == 3 ? '' : 'btn-outline'" class="btn btn-square btn-xs btn-warning text-md font-semibold" @click="activeLevel = 3">4</button>
          <button :class="activeLevel == 4 ? '' : 'btn-outline'" class="btn btn-square btn-xs btn-warning text-md font-semibold" @click="activeLevel = 4">5</button>
          <button :class="activeLevel == 5 ? '' : 'btn-outline'" class="btn btn-square btn-xs btn-warning text-md font-semibold" @click="activeLevel = 5">6</button>
        </div>
      </div>
    </div>
  </div>
</template>
