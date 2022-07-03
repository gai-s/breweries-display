<template>
    <header>
        <h1>{{title}} Dispay</h1>
        <Button 
        :text="showCreateForm ? 'close': 'add new'" 
        :color="showCreateForm ? 'LightSkyBlue': 'purple'" 
        @btn-click="$emit('toggle_create_form')"/>
    </header>
</template>

<script>
    import Button from './Button'
    export default{
        name: 'Header',
        props: {
            title: String,
            showCreateForm: Boolean
        },
        components: {
            Button,
        },
        emits: ['toggle_create_form']
    }
</script>

<style scoped>
    header{
        display: flex;
        justify-content: space-between;
        text-align: center;
        margin-bottom: 20px;
    }

</style>