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

      <div v-if="!showCart">

        <div class="w-full flex justify-center" v-if="loading">
          <iframe class="w-1/2" src="https://embed.lottiefiles.com/animation/99297" title="loading"></iframe>
        </div>

        <div class="w-full flex justify-center" v-else-if="error">
          <iframe class="w-1/2" src="https://embed.lottiefiles.com/animation/95614" title="error"></iframe>
        </div>

        <Lessons :lessons="sortedLessons" @add-item-to-cart="addToCart" v-else />

      </div>

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
    shoppingCart: [],
    searchQuery: '',
    lessons: [],
    loading: false,
    error: null,
    showCart: false,
    sortBy: 'subject',
    orderBy: 'asc',
    url: 'https://course.eu-north-1.elasticbeanstalk.com',
    sortOptions: ["subject", "location", "price", "spaces"],
    orders: [
      {
        text: "Ascending",
        value: "asc",
      },
      {
        text: "Descending",
        value: "desc",
      },
    ],
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
    capitalize(str) {
      return str.charAt(0).toUpperCase() + str.slice(1);
    },
    getSortValue(lesson) {
      switch (this.sortBy) {
        case "subject":
        case "location":
          return lesson[this.sortBy].toLowerCase();
        case "price":
        case "availability":
          return lesson[this.sortBy];
        default:
          return 0;
      }
    },
    compareValues(a, b) {
      return a > b ? 1 : a < b ? -1 : 0;
    },
    addToCart(lesson) {
      if (lesson.spaces > 0) {
        // Add the lesson to the cart
        this.shoppingCart.push({
          id: lesson._id,
          subject: lesson.subject,
          price: lesson.price,
          location: lesson.location,
          spaces: 1,
        });

        // Reduce the remaining space by one
        lesson.spaces -= 1;
      }
    },
    toggleCart() {
      // Toggle the visibility of the shopping cart
      this.showCart = !this.showCart;
    },
  },
  computed: {
    sortedLessons() {
      return this.lessons
        .sort((a, b) => {
          const order = this.orderBy === 'asc' ? 1 : -1;
          return a[this.sortBy] > b[this.sortBy] ? order : -order;
        });
    },
  },
  created() {
    this.getLessons();
  },
}

</script>