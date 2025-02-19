<template>
    <div class="m-8">
        <UHorizontalNavigation :links="links" class="border-b border-gray-200 dark:border-gray-800" />
        <store v-if="mode == 0"/>
    </div>
</template>

<script setup>
    const links = [{
    label: 'Home',
    icon: 'i-heroicons-home',
    to: '/'
    }, 
]
const mode = ref(0);
</script>

<style scoped>
#disabled {
    opacity: 0.5;
    pointer-events: none;
    user-select: none;
}
.store-content {
    display: flex;
    flex-direction: column;
    /* max-width: 720px; */
}
.store-item {
    border-radius: 10px;
}
.store-item input, select {
    width: 50%;
}
select{
    padding: 10px;
    border-radius:50px;
    background-color: inherit;
    color:var(--color-text);
    border:1px solid var(--color-background-mute)
}
</style>