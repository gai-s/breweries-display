<template>
  <div class="container">
    <Header title="Breweries" :showCreateForm="showCreateForm" @toggle_create_form="toggleCreateForm()"/>
    <div v-show="showCreateForm">
      <CreateItem @add-item="addItem" @edit-item="UpdateItem" :editMode="editMode" :curEditItem="curEditItem" :curItemState="curItemState"/>
    </div>
    <Items @deleteItem="deleteItem" @startEditItem="startEditItem" :items="items" />
  </div>
</template>

<script>
'use strict'
import Header from './components/Header'
import CreateItem from './components/CreateItem.vue'
import Items from './components/Items'

export default {
  name: 'App',
  components: {
    Header,
    CreateItem,
    Items
  },
  data() {
    return{
      mainObj: {},
      items: [],
      showCreateForm: true,
      editMode: false,
      curEditItem: {},
      curItemState: '',
      stateIndex: -1,
      breweryIndex: -1
    }
  },   
  methods: {
    async fetchItems() {
      const res = await fetch(`https://api.openbrewerydb.org/breweries`)
      const data = await res.json()
      let unique_states=[...new Set(data.map((i)=>i.state))]
      const mainObject = {
            states: {}
      }
      for(let cur_state of unique_states){
          mainObject.states[`${cur_state}`] = {stateName: `${cur_state}`,
                                            breweries: {}
                                          }
      }
      for(let cur_item of data){
        mainObject.states[`${cur_item.state}`].breweries[`${cur_item.id}`]={city: `${cur_item.city}`,
                                                      street: `${cur_item.street}`, name: `${cur_item.name}`}
      }
      console.log(mainObject)
      this.mainObj= mainObject
      return
    },

    addItem(item){
      let locate_state = this.binarySearch(this.items, 0, this.items.length-1, item, this.statesCompare)
      let state_position = locate_state.pos
      if(locate_state.res){
        let locate_brewery = this.binarySearch(this.items[state_position].breweries, 0, this.items[state_position].breweries.length-1, {id: item.id, details: item}, this.breweriesCompare)
        //In case this brewery name already exist in this city, the item will not be added
        if (locate_brewery.res){
          alert(`A Brewery named ${item.name} already exist in ${item.city}, ${item.state}, No save accure`)
          return 0
          }
        //In case the brewery's state already exist, the item will be added to the right state array
        else{
          let brewery_position = locate_brewery.pos
          let new_brewery = {city: item.city, street: item.street, name: item.name}
          this.mainObj.states[item.state].breweries[item.id]=new_brewery
          this.items[state_position].breweries.splice(brewery_position,0,{id: item.id, details: new_brewery})
        }
      }
      //In case the brewery's state do not exist, a new state will be added to the data along with new brewery
      else{
          let new_brewery = {city: `${item.city}`, street: `${item.street}`, name: `${item.name}`}
          this.mainObj.states[item.state]={stateName: item.state, breweries: {}}
          this.mainObj.states[item.state].breweries[item.id]=new_brewery
          this.items.splice(state_position,0,{state: `${item.state}`, breweries: [{id: `${item.id}`, details: new_brewery}]})
      }
      console.log("mainObj arr is: ", this.mainObj)
      console.log("items arr is: ", this.items)
      return 1
    },
    
    deleteItem(id, i, j){
      if(confirm('Are you sure?')){
        //let index = this.items.findIndex(cur => cur.stateName===state_name)
        let cur_state = this.items[i]
        delete(this.mainObj.states[cur_state.state].breweries[id])
        this.items[i].breweries.splice(j,1)
        //let cur_state=this.items[index].breweries.filter((cur)=>cur.id==id)
      }
    },

    startEditItem(item, i, j){
      //already on editMode and on the same item will exit from edit mode
      if(this.editMode && this.curEditItem.id===item.id){
        this.editMode=false
        this.curEditItem=null
      }
      //Otherwise wil enter editMode
      else{
        this.editMode=true
        this.curEditItem=item
        this.curItemState = this.items[i].state
        this.stateIndex = i
        this.breweryIndex = j
        this.showCreateForm=true
      }
    },

    UpdateItem(item){
          if(item.state.toUpperCase()==this.curItemState.toUpperCase()){
            let brewery_location=this.items[this.stateIndex].breweries[this.breweryIndex].details
            brewery_location.street=item.street
            //In case the city or name had changed the item might change his location in sorted breweries array
            if(brewery_location.city!=item.city || brewery_location.name!=item.name){
              let new_locate_brewery = this.binarySearch(this.items[this.stateIndex].breweries, 0, this.items[this.stateIndex].breweries.length-1, {id: item.id, details: item}, this.breweriesCompare)
              //check for duplication of changed item, if so no change accure
              if(new_locate_brewery.res){
                  alert(`A Brewery named ${item.name} already exist in ${item.city}, ${item.state}, No saving accure`)
                  brewery_location.street=this.curEditItem.street

              }
              //No duplication, item will be update and place in the right sorted position
              else{
                brewery_location.city=item.city
                brewery_location.name=item.name
                this.items[this.stateIndex].breweries.splice(this.breweryIndex,1)
                //position fix after erasing 
                let new_position = this.breweryIndex<new_locate_brewery.pos ? (new_locate_brewery.pos-1) : new_locate_brewery.pos
                this.items[this.stateIndex].breweries.splice(new_position,0,{id: item.id, details: brewery_location})
              }
            }
            this.editMode=false
            this.curEditItem=null

          }
          //In case of the brewery's state is updated
          else{
              if(this.addItem(item)){
                delete(this.mainObj.states[this.curItemState].breweries[item.id])
                //position fix after insertion (in case a new state has inserted)
                this.stateIndex = this.items[this.stateIndex].name==this.curItemState ? this.stateIndex: this.stateIndex+1
                this.items[this.stateIndex].breweries.splice(this.breweryIndex,1)
                this.editMode=false
                this.curEditItem=null
              }

          }
          console.log("mainObj arr is: ", this.mainObj)
          console.log("items arr is: ", this.items)
    },

    toggleCreateForm(){
      this.showCreateForm = !this.showCreateForm
    },
    //Inner fucntion of binary search. return an object with result and position of value if exist
    //in case value does not exist in array, return the position to be inserted in.  
    binarySearch(arr, low, high, value, compare_func){
      if(high < low)
        return {res: false, pos: low}
      let mid = Math.trunc((low + high) / 2)
      if (!compare_func(arr[mid], value))
        return {res: true, pos: mid}
      if (compare_func(arr[mid], value)<0 )
        return this.binarySearch(arr, mid+1, high, value, compare_func)
      return this.binarySearch(arr, low, mid-1, value, compare_func)
    },

    statesCompare(a,b){
      return (a.state.toLowerCase().localeCompare(b.state.toLowerCase()))
    },
    
    breweriesCompare(a,b){
            if(a.details.city.toLowerCase()==b.details.city.toLowerCase())
              return a.details.name.toLowerCase().localeCompare(b.details.name.toLowerCase())
            return a.details.city.toLowerCase().localeCompare(b.details.city.toLowerCase())
    }
  },

  async created() {
    await this.fetchItems()
    this.items = Object.keys(this.mainObj.states).map(cur_state=>{return {state: cur_state, breweries:[]}})
    let i=0
    let cur={}
    for(let cur_state in this.mainObj.states){
      for (let key in this.mainObj.states[cur_state].breweries){
        this.items[i].breweries.push({id: key, details: this.mainObj.states[cur_state].breweries[key]})
      }
      this.items[i].breweries.sort(this.breweriesCompare)
      i++
    }
    this.items.sort(this.statesCompare)

    this.showCreateForm = true,
    this.editMode = false,
    this.curEditItem= {}
  },
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400&display=swap');
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
body {
  font-family: 'Poppins', sans-serif;
}
.container {
  max-width: 500px;
  margin: 30px auto;
  overflow: auto;
  min-height: 300px;
  border: 1px solid steelblue;
  padding: 30px;
  border-radius: 5px;
}
.btn {
  display: inline-block;
  min-width: 110px;
  background: #000;
  color: #fff;
  border: none;
  padding: 10px 20px;
  margin: 5px;
  border-radius: 5px;
  cursor: pointer;
  text-decoration: none;
  font-size: 15px;
  font-family: inherit;
}
.btn:focus {
  outline: none;
}
.btn:active {
  transform: scale(0.98);
}
.btn-block {
  display: block;
  width: 100%;
  margin: auto;
  margin-top: 10px;
}
.hidden{
  display: none;
}
</style>
