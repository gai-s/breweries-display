<template>
    <div v-for="(item, i) in stateBreweries" :key="i">
        <Item :item="item" :state_name="state_name" :state_key="state_key" :brewery_key="i" @delete-item="deleteItem"  @start-edit-item="startEditItem"/>
    </div>
</template>

<script>
    import Item from './Item.vue'
    export default {
         name: 'State',
         props:{
            stateBreweries: Array,
            state_name: String,
            state_key: Number
         },
         components:{
            Item
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