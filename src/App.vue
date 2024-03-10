<template>
  <main class="bg-gray-100">
    <div class="px-6 mx-auto max-w-full lg:px-12 py-4">

      <div class="flex justify-end mb-8">

        <!-- "Shopping Cart" button -->
        <button @click="toggleCart" :disabled="shoppingCart.length === 0"
          class="bg-blue-500 text-white py-2 px-4 rounded hover:bg-blue-700 focus:outline-none focus:ring focus:border-blue-300 disabled:hover:bg-blue-500 disabled:opacity-50">
          {{ showCart ? "Hide Cart" : "Show Cart" }}
        </button>

      </div>

      <Lessons v-if="!showCart" />

      <Checkout v-else class="mt-8" />
    </div>
  </main>
</template>

<script>
import Lessons from './components/Lessons.vue';
import Checkout from './components/Checkout.vue';

export default {
  components: {
    Lessons,
    Checkout
  },
  data: () => ({
    loading: false,
    error: null,
    url: 'https://course.eu-north-1.elasticbeanstalk.com',
    showCart: false,
    shoppingCart: [],
    searchQuery: '',
    lessons: [],
  }),
  methods: {
    async getLessons() {
      try {
        this.loading = true;

        const url = `${this.url}/lessons/?search=${this.searchQuery}`;

        const response = await fetch(url);

        this.lessons = await response.json();
      } catch (error) {
        this.error = error;
      } finally {
        this.loading = false;
      }
    },
    toggleCart() {
      // Toggle the visibility of the shopping cart
      this.showCart = !this.showCart;
    },
  },
  created() {
    this.getLessons();
  },
}

</script>