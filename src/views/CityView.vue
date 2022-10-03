<template>
    <div>
        <Suspense>
            <AsyncCityView />
            <template #fallback>
                <p class="flex flex-col items-center text-white py-12 text-4xl mb-2">
                    Loading...
                </p>
            </template>
        </Suspense>
    </div>
</template>

<script setup>
import AsyncCityView from '../components/AsyncCityView.vue';

</script>