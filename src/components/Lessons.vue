<template>
    <div
        id="lessons-list"
        class="grid grid-cols-1 md:grid-cols-3 lg:grid-cols-4 gap-8"
    >
        <div
            v-for="lesson in lessons"
            :key="lesson.id"
            class="bg-white rounded-lg shadow-md overflow-hidden p-4 border-l-4 border-blue-500 mb-4"
        >
            <img
                :src="`./imgs${lesson.image}`"
                :alt="lesson.subject"
                class="w-full h-48 object-cover object-center mb-4"
            />
            <h2 class="text-lg font-semibold mb-2 text-blue-500">
                <strong>Subject:</strong> {{ lesson.subject }}
            </h2>
            <p class="text-gray-600 mb-2">
                <strong>Location:</strong> {{ lesson.location }}
            </p>
            <p class="text-gray-800">
                <strong>Price:</strong> £{{ lesson.price }}
            </p>
            <p class="text-gray-600">
                <strong>Spaces:</strong> {{ lesson.spaces }}
            </p>

            <!-- "Add to Cart" button -->
            <button
                @click="$emit('add-item-to-cart', lesson)"
                :disabled="lesson.spaces === 0"
                class="mt-2 bg-blue-500 text-white py-2 px-4 rounded hover:bg-blue-700 focus:outline-none focus:ring focus:border-blue-300 disabled:opacity-50"
            >
                Add to Cart
            </button>
        </div>
    </div>
</template>

<script>
export default {
    props: ['lessons'],
}
</script>