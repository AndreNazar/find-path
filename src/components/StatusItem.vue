<template>
    <div class="status_item">
        <p>Путь №{{path.id}}</p>
        <p>Шагов: {{path.steps}}</p>
        <p>Время: {{ path.time }}ms</p>
    </div>
</template>

<script>

    export default {
        name: "StatusItem",

        props: {
            path: Object,
        },
    }
</script>

<style lang="scss" scoped>

</style>