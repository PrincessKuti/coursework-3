<template>
    <main class="bg-gray-100">
        <div class="px-6 mx-auto max-w-full lg:px-12 py-4">
            <div class="flex justify-end mb-8">
                <!-- "Shopping Cart" button -->
                <button
                    @click="toggleCart"
                    :disabled="shoppingCart.length === 0"
                    class="bg-blue-500 text-white py-2 px-4 rounded hover:bg-blue-700 focus:outline-none focus:ring focus:border-blue-300 disabled:hover:bg-blue-500 disabled:opacity-50"
                >
                    {{ showCart ? 'Hide Cart' : 'Show Cart' }}
                </button>
            </div>

            <div v-if="!showCart">
                <!-- Search -->
                <div class="mb-8">
                    <input
                        v-model="searchQuery"
                        type="text"
                        placeholder="Search lessons"
                        class="p-2 block w-full border rounded-md shadow-sm focus:outline-none focus:ring focus:border-blue-300"
                    />
                </div>

                <!-- Sort Options -->
                <div
                    class="flex flex-col space-y-2 sm:space-y-0 sm:flex-row sm:justify-between mb-8"
                >
                    <div class="flex flex-wrap">
                        <label
                            v-for="option in sortOptions"
                            :key="option"
                            :for="`sortAttribute${option}`"
                            class="inline-flex items-center space-x-2 cursor-pointer text-lg mr-4 mb-2"
                        >
                            <input
                                type="radio"
                                :id="`sortAttribute${option}`"
                                v-model="sortBy"
                                :value="option"
                                class="form-radio text-blue-500 focus:ring-2 focus:ring-blue-300"
                            />
                            <span>{{ capitalize(option) }}</span>
                        </label>
                    </div>
                    <div class="flex flex-wrap">
                        <label
                            v-for="order in orders"
                            :key="order.value"
                            :for="`sortOrder${order.value}`"
                            class="inline-flex items-center space-x-2 cursor-pointer text-lg mr-4 mb-2"
                        >
                            <input
                                type="radio"
                                :id="`sortOrder${order.value}`"
                                v-model="orderBy"
                                :value="order.value"
                                class="form-radio text-blue-500 focus:ring-2 focus:ring-blue-300"
                            />
                            <span>{{ order.text }}</span>
                        </label>
                    </div>
                </div>

                <div
                    class="w-full flex justify-center"
                    v-if="loading && !sortedLessons"
                >
                    <iframe
                        class="w-1/2"
                        src="https://embed.lottiefiles.com/animation/99297"
                        title="loading"
                    ></iframe>
                </div>

                <div class="w-full flex justify-center" v-else-if="error">
                    <iframe
                        class="w-1/2"
                        src="https://embed.lottiefiles.com/animation/95614"
                        title="error"
                    ></iframe>
                </div>

                <Lessons
                    :lessons="sortedLessons"
                    @add-item-to-cart="addToCart"
                    v-else
                />
            </div>

            <Checkout
                v-else
                :shopping-cart="shoppingCart"
                :order-submitted="orderSubmitted"
                :is-valid-checkout="isValidCheckout"
                :checkout-data="checkoutData"
                :loading="loading"
                @remove-item-from-cart="removeFromCart"
                @checkout="checkout"
                class="mt-8"
            />
        </div>
    </main>
</template>

<script>
import Lessons from './components/Lessons.vue'
import Checkout from './components/Checkout.vue'

export default {
    components: {
        Lessons,
        Checkout,
    },
    data: () => ({
        shoppingCart: [],
        searchQuery: '',
        lessons: [],
        loading: false,
        error: null,
        showCart: false,
        orderSubmitted: false,
        sortBy: 'subject',
        orderBy: 'asc',
        url: 'https://course.eu-north-1.elasticbeanstalk.com',
        checkoutData: {
            name: '',
            phone: '',
        },
        sortOptions: ['subject', 'location', 'price', 'spaces'],
        orders: [
            {
                text: 'Ascending',
                value: 'asc',
            },
            {
                text: 'Descending',
                value: 'desc',
            },
        ],
    }),
    methods: {
        async getLessons() {
            try {
                this.loading = true

                const url = `${this.url}/lessons/?search=${this.searchQuery}`

                const response = await fetch(url)

                this.lessons = await response.json()
            } catch (error) {
                this.error = error
            } finally {
                this.loading = false
            }
        },
        async createNewOrder(order) {
            try {
                this.loading = true

                const url = `${this.url}/orders`

                const response = await fetch(url, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(order),
                })
            } catch (error) {
                this.error = error
            } finally {
                this.loading = false
            }
        },
        async updateLesson(lesson) {
            try {
                this.loading = true

                console.log(lesson)

                const url = `${this.url}/orders/lessons/${lesson.id}`

                const response = await fetch(url, {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        lesson,
                    }),
                })
            } catch (error) {
                this.error = error
            } finally {
                this.loading = false
            }
        },
        getSortValue(lesson) {
            switch (this.sortBy) {
                case 'subject':
                case 'location':
                    return lesson[this.sortBy].toLowerCase()
                case 'price':
                case 'availability':
                    return lesson[this.sortBy]
                default:
                    return 0
            }
        },
        compareValues(a, b) {
            return a > b ? 1 : a < b ? -1 : 0
        },
        capitalize(str) {
            return str.charAt(0).toUpperCase() + str.slice(1)
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
                })

                // Reduce the remaining space by one
                lesson.spaces -= 1
            }
        },
        removeFromCart(cartItem) {
            // Remove from cart logic
            const index = this.shoppingCart.findIndex(
                (item) => item.id === cartItem.id
            )

            if (index !== -1) {
                this.shoppingCart.splice(index, 1)
            }

            // Add the removed lesson back to the lesson list
            const lessonIndex = this.lessons.findIndex(
                (lesson) => lesson._id === cartItem.id
            )

            if (lessonIndex !== -1) {
                this.lessons[lessonIndex].spaces += cartItem.spaces
            }

            // Check if the cart is empty after removing an item
            this.showCart = this.shoppingCart.length > 0
        },
        toggleCart() {
            // Toggle the visibility of the shopping cart
            this.showCart = !this.showCart
        },
        checkout() {
            // checkout form is not valid
            if (!this.isValidCheckout) {
                return
            }

            this.shoppingCart.forEach(async (item) => {
                await this.createNewOrder({
                    name: this.checkoutData.name,
                    phone_number: this.checkoutData.phone,
                    lesson_id: item.id,
                    spaces: item.spaces,
                })

                await this.updateLesson(item)
            })

            // Display a message confirming the order submission
            this.orderSubmitted = true

            this.checkoutData.name = ''
            this.checkoutData.phone = ''

            setTimeout(() => {
                this.toggleCart()

                this.shoppingCart = []
            }, 2000)
        },
    },
    computed: {
        sortedLessons() {
            return this.lessons.sort((a, b) => {
                const order = this.orderBy === 'asc' ? 1 : -1
                return a[this.sortBy] > b[this.sortBy] ? order : -order
            })
        },
        isValidCheckout() {
            // Check if name and phone are valid
            const nameRegex = /^[a-zA-Z\s]+$/
            const phoneRegex = /^[+0-9()\- ]+$/
            return (
                nameRegex.test(this.checkoutData.name) &&
                phoneRegex.test(this.checkoutData.phone)
            )
        },
    },
    created() {
        // register service worker
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker
                    .register('./service-worker.js', {
                        scope: '/coursework-3/',
                    })
                    .then((registration) => {
                        console.log('Service worker registered: ', registration)
                    })
                    .catch((error) => {
                        console.log(
                            'Service worker registration failed: ',
                            error
                        )
                    })
            })
        }

        this.getLessons()
    },
}
</script>