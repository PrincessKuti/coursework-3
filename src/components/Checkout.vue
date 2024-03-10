<template>
    <div class="mt-8">
        <h2 class="text-2xl font-semibold mb-4">Shopping Cart</h2>
        <ul>
            <li v-for="cartItem in shoppingCart" :key="cartItem.id"
                class="bg-white rounded-lg shadow-md overflow-hidden p-4 border-l-4 border-blue-500 mb-4 flex justify-between items-center">
                <div>
                    <p class="text-lg font-semibold text-blue-500">{{ cartItem.subject }}</p>
                    <p class="text-gray-600">{{ cartItem.spaces }} spaces</p>
                </div>
                <button @click="$emit('remove-item-from-cart', cartItem)"
                    class="bg-red-500 text-white py-2 px-4 rounded hover:bg-red-700 focus:outline-none focus:ring focus:border-red-300">
                    Remove
                </button>
            </li>
        </ul>

        <!-- Checkout Section -->
        <div class="mt-8">
            <h2 class="text-2xl font-semibold mb-4">Checkout</h2>

            <!-- Error Alert -->
            <div v-if="!isValidCheckout" class="bg-red-100 border border-red-400 text-red-700 px-4 py-2 rounded mb-4">
                <strong>Error:</strong> Please provide a valid name and phone number.
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                    <label for="name" class="block text-sm font-medium text-gray-600">Name</label>
                    <input v-model="checkoutData.name" type="text" id="name" name="name"
                        class="mt-1 p-2 block w-full border rounded-md shadow-sm focus:outline-none focus:ring focus:border-blue-300">
                </div>
                <div>
                    <label for="phone" class="block text-sm font-medium text-gray-600">Phone</label>
                    <input v-model="checkoutData.phone" type="text" id="phone" name="phone"
                        class="mt-1 p-2 block w-full border rounded-md shadow-sm focus:outline-none focus:ring focus:border-blue-300">
                </div>
            </div>

            <button @click="$emit('checkout')" :disabled="!isValidCheckout"
                class="mt-4 bg-green-500 text-white py-2 px-4 rounded hover:bg-green-700 focus:outline-none focus:ring focus:border-green-300">
                <div v-if="loading" class="flex h-5 w-5 m-auto relative">
                    <span
                        class="animate-ping absolute inline-flex h-full w-full rounded-full bg-primary opacity-75"></span>
                    <span class="inline-flex rounded-full h-5 w-5 bg-primary-dark"></span>
                </div>
                <span v-else>Checkout</span>
            </button>
            <p v-if="orderSubmitted" class="mt-4 text-green-500">Order submitted successfully!</p>
        </div>
    </div>
</template>

<script>
export default {
    props: ['shoppingCart', 'isValidCheckout', 'orderSubmitted', 'checkout-data', 'loading'],
}
</script>