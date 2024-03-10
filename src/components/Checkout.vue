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

        
    </div>
</template>

<script>
export default {
    props: ['shoppingCart', 'isValidCheckout', 'orderSubmitted'],
}
</script>