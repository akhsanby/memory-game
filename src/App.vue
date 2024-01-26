<script setup>
import { onMounted, ref } from "vue";
import * as solidIcons from "@fortawesome/free-solid-svg-icons";
import * as regularIcons from "@fortawesome/free-regular-svg-icons";
import * as brandsIcons from "@fortawesome/free-brands-svg-icons";

const items = [
  // {
  //   name: "Ayaka",
  //   image: "/images/Ayaka.png",
  // },
  // {
  //   name: "Dehya",
  //   image: "/images/Dehya.png",
  // },
  // {
  //   name: "Eula",
  //   image: "/images/Eula.png",
  // },
  // {
  //   name: "Furina",
  //   image: "/images/Furina.png",
  // },
  // {
  //   name: "Ganyu",
  //   image: "/images/Ganyu.png",
  // },
  // {
  //   name: "Hutao",
  //   image: "/images/Hutao.png",
  // },
  // {
  //   name: "Kokomi",
  //   image: "/images/Kokomi.png",
  // },
  // {
  //   name: "Nahida",
  //   image: "/images/Nahida.png",
  // },
  // {
  //   name: "Navia",
  //   image: "/images/Navia.png",
  // },
  // {
  //   name: "Nilou",
  //   image: "/images/Nilou.png",
  // },
  // {
  //   name: "Shenhe",
  //   image: "/images/Shenhe.png",
  // },
  // {
  //   name: "Shougun",
  //   image: "/images/Shougun.png",
  // },
  // {
  //   name: "Yae",
  //   image: "/images/Yae.png",
  // },
  // {
  //   name: "Yelan",
  //   image: "/images/Yelan.png",
  // },
  // {
  //   name: "Yoimiya",
  //   image: "/images/Yoimiya.png",
  // },
];

const currentTime = ref();
const resultTime = ref();
const movesCount = ref(0);
const matchCount = ref(0);
const cardValues = ref();
const firstCard = ref();
const firstCardName = ref();
const secondCard = ref();
const secondCardName = ref();

function getRandomIcon() {
  const iconArray = Object.values({ ...solidIcons, ...regularIcons, ...brandsIcons });
  const randomIndex = Math.floor(Math.random() * iconArray.length);
  return iconArray[randomIndex];
}

function timeGeneratorCustom(sec = 90) {
  sec = Math.max(0, Math.floor(sec));

  const intervalId = setInterval(() => {
    if (sec > 0) {
      sec--;

      const minutes = Math.floor((sec % 3600) / 60)
        .toString()
        .padStart(2, "0");
      const seconds = (sec % 60).toString().padStart(2, "0");

      resultTime.value = `${minutes} : ${seconds}`;
      currentTime.value = sec;
    } else {
      clearInterval(intervalId);
      resultTime.value = null;
      cardValues.value = null;
    }
  }, 1000);
}

//For calculating moves
function movesCounter() {
  movesCount.value += 1;
}

const generateRandom = (size = 4) => {
  for (let index = 0; index < size * size; index++) {
    const randomIcon = getRandomIcon();
    items.push({ name: randomIcon.iconName, icon: randomIcon });
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
      } else {
        let [tempFirst, tempSecond] = [firstCard.value, secondCard.value];
        setTimeout(() => {
          tempFirst.classList.remove("flipped");
          tempSecond.classList.remove("flipped");
        }, 600);
      }
      firstCard.value = false;
      secondCard.value = false;
    }
  }
}

function startGame() {
  timeGeneratorCustom(60);
}

onMounted(() => {
  const randomCard = generateRandom();
  cardValues.value = [...randomCard, ...randomCard];
});
</script>

<template>
  <div class="w-screen h-screen flex justify-center items-center">
    <div class="card bg-white shadow-xl rounded-xl">
      <div v-if="currentTime >= 0 && cardValues" class="stats-container flex flex-col justify-center items-center mt-[1.2rem] font-semibold text-black">
        <div id="moves-count"><span>Moves:</span> {{ movesCount }}</div>
        <div id="time" class="text-xl">{{ resultTime }}</div>
      </div>
      <div class="card-body grid grid-cols-4 gap-3" id="game-container">
        <div v-if="currentTime >= 0 && cardValues" v-for="card in cardValues" class="card-container card w-[6rem] h-[6rem]">
          <div @click.self="(e) => handleClick(e, card.name)" class="card-before bg-warning rounded-xl border-4 border-black shadow-xl flex justify-center card-title text-[3rem] w-full h-full absolute backface-visibility-hidden text-black rotate-y-0">?</div>
          <div class="card-after bg-white rounded-xl border-4 border-black shadow-xl flex justify-center card-title text-[3rem] w-full h-full absolute backface-visibility-hidden text-black rotate-y-180">
            <font-awesome-icon :icon="[card.icon.prefix, card.icon.iconName]" />
          </div>
        </div>
        <div v-else class="controls-container col-span-4 flex flex-col justify-center items-center space-y-4">
          <p class="text-black font-semibold text-xl">Welcome to Memory Game</p>
          <button class="btn btn-warning" @click="startGame">Start Game</button>
        </div>
      </div>
    </div>
  </div>
</template>
