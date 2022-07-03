<template>
    <div v-for="(state, i) in items" :key="i" :class="[!state.breweries.length ? 'hidden' : '']">
        <h3 class= 'state-headline'>{{state.state}}</h3>
        <State :stateBreweries="state.breweries" :state_name="state.state" :state_key="i" @delete-item="deleteItem"  @start-edit-item="startEditItem"/>
    </div>
</template>

<script>
    import State from './State.vue'
    export default{
        name: 'Items',
        props: {
            items: Array,
        },
        components:{
            State
        },
        emits: ['deleteItem', 'startEditItem'],
        methods:{
            deleteItem(id, state_key, brewery_key){
                this.$emit('deleteItem', id, state_key, brewery_key)
            },
            
            startEditItem(item, state_key, brewery_key){
                this.$emit('startEditItem', item, state_key, brewery_key)
            }
        } 

    }
</script>

<style scope>
.state-headline{
    color: purple;
}
</style>