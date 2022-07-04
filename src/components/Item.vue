<template>
    <div :class="[(state_name=='New York') ? 'new_york': '', 'card']">
        <h3>
            <span>{{item.details.name}}</span>
            <i class="fas fa-times" @click="$emit('delete-item', item.id, state_key, brewery_key)"></i>
            <i class="fas fa-edit" @click="$emit('start-edit-item', item, state_key, brewery_key)"></i>
        </h3>
        <p>{{((item.details.street!=='null')&&(item.details.street!=="")&&(item.details.street!=undefined)) ? (item.details.street + ', ') : ''}} {{item.details.city}}, {{state_name}}</p>
    </div>
</template>

<script>
import { capitalize } from '@vue/shared'
    export default {
         name: 'Item',
         props:{
            item: Object,
            state_name: String,
            state_key: Number,
            brewery_key: Number
         },
        emits: ['delete-item', 'start-edit-item']
    }
</script>

<style scope>
i{
    color:lightblue;
    margin-right: 5px;
    margin-top: 5px;
    cursor: pointer;
    align-self: flex-start;
}
.fa-times{
    margin-left: auto;

}
.fas:hover {
  transform: scale(0.95);
  opacity: 0.8;
}
.fas:active {
  transform: scale(0.98);
}

.card {
  background: #f4f4f4;
  margin: 5px;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 5px;
}

.card>h3 {
  display: flex;
}

.new_york{
  border-left: 5px solid #008B8B;
}
</style>