<script setup>
import { onMounted } from "vue";





onMounted(() => {
  const cardContainer = document.querySelector(".card-container");
  cardContainer.innerHTML += `
  
    <div class="card-after"><img src="" /></div>
  `;
});
</script>

<template>
  <div class="card w-[6rem] h-[6rem] border-4 border-black bg-warning shadow-xl rounded-xl">
    
      <!-- <div class="card-before text-black text-[3rem] flex justify-center items-center">?</div> -->
      <!-- <div class="card-after">
        <img src="" />
      </div> -->
    </div>
  </div>
</template>
