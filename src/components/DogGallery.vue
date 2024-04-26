<script setup>
import axios from "axios";
import { ref, computed, watch } from "vue";

const currentPage = ref(1);
const allImages = ref([]);
const selectedBreed = ref(null);
const dogBreeds = ref([]);

async function getDogBreeds() {
  const response = await axios.get("https://dog.ceo/api/breeds/list/all");
  dogBreeds.value = Object.keys(response.data.message);
  selectedBreed.value = dogBreeds.value[0];
}

getDogBreeds();

async function getDogImages() {
  if (!selectedBreed.value) return;
  const response = await axios.get(
    `https://dog.ceo/api/breed/${selectedBreed.value}/images`
  );
  allImages.value = response.data.message;
}

watch(selectedBreed, getDogImages);

const startIndex = computed(() => (currentPage.value - 1) * 12);
const endIndex = computed(() =>
  Math.min(startIndex.value + 12, allImages.value.length)
);

const dogImages = computed(() =>
  allImages.value.slice(startIndex.value, endIndex.value)
);
</script>

<template>
  <main>
    <h1>DOG GALLERY</h1>
    <div>
      <p>Choose a breed of a dog</p>
      <select v-model="selectedBreed">
        <option v-for="breed in dogBreeds" :key="breed" :value="breed">
          {{ breed }}
        </option>
      </select>
    </div>
    <Suspense>
      <template #default>
        <div class="dog-grid">
          <div v-for="image in dogImages" :key="image" class="dog-image">
            <img :src="image" alt="Dog Image" />
          </div>
        </div>
      </template>
      <template #fallback>
        <div>
          <p>Loading...</p>
        </div>
      </template>
    </Suspense>
    <div class="pagination">
      <button @click="currentPage--" :disabled="currentPage === 1">
        Previous
      </button>
      <span>Page {{ currentPage }}</span>
      <button @click="currentPage++" :disabled="endIndex >= allImages.length">
        Next
      </button>
    </div>
  </main>
</template>

<style scoped>
main {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-bottom: 2rem;
  text-align: center;
  background-color: steelblue;
  border-radius: 2rem;
  color: white;
}

.dog-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-gap: 2rem;
  margin-top: 2rem;
  margin-bottom: 2rem;
  padding: 2rem;
  background-color: black;
  border-radius: 2rem;
}

.dog-image {
  width: 100%;
}

.dog-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 1rem;
}

.pagination {
  display: flex;
  align-items: center;
  border-radius: 1.5rem;
}

button {
  padding: 1rem;
  border-radius: 1.5rem;
  color: aliceblue;
  background-color: black;
  border: none;
}

button:disabled {
  opacity: 0.7;
}

span {
  margin-left: 1rem;
  margin-right: 1rem;
}

select {
  padding: 1rem;
  border-radius: 1.5rem;
}
</style>
