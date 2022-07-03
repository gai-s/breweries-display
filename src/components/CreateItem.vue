<template>
    <form class="create-form" @submit='onSubmit'>
        <div class="form-control">
            <label for="name">
                Name: 
            </label>
            <input type="text" name="name" v-model="name" id="name" placeholder="Add new Item name" />
        </div>

        <div class="form-control">
            <label for="state">
                state: 
            </label>
            <input type="text" name="state" v-model="state" id="state" placeholder="Add state" />
        </div>

        <div class="form-control">
            <label for="city">
                city: 
            </label>
            <input type="text" name="city" v-model="city" id="city" placeholder="Add city" />
        </div>

        <div class="form-control">
            <label for="street">
                street: 
            </label>
            <input type="text" name="street" v-model="street" id="street" placeholder="Add street" />
        </div>


        <input type="submit" :value="[(editMode) ? 'update' : 'save']" @click="onSubmit" class="btn btn-block"/>

    </form>
</template>

<script>  
    export default{
        name: 'CreateItem',
        props: {
            editMode: Boolean,
            curEditItem: Object,
            curItemState: String
        },
        data () {
            return{
                id: '',
                state: '',
                city: '',
                street: '',
                name: ''
            }
        },  
        watch:{
                curEditItem:{
                    handler(newval, oldval){
                        if(this.editMode){
                            this.id=this.curEditItem.id
                            this.name=this.curEditItem.details.name
                            this.state=this.curItemState
                            this.city=this.curEditItem.details.city
                            this.street=(this.curEditItem.details.street!='null' ? this.curEditItem.details.street: '')
                        }
                        if(!this.editMode){
                            this.id=''
                            this.state=''
                            this.city=''
                            this.street=''
                            this.name=''

                        }

                    },
                    immediate: true
                }
                },
        methods:{
            onSubmit(e){
                e.preventDefault();
                //validation and existence check
                if(!this.name){
                    alert('Please add name')
                    return;
                }
                if(!this.state){
                    alert('Please add a state')
                    return;
                }
                if(!this.city){
                    alert('Please add a city')
                    return;
                }

                const newItem={
                    id : this.id,
                    state : this.capitalizeEachWord(this.state),
                    city : this.capitalizeEachWord(this.city),
                    street : this.street,
                    name: this.capitalizeEachWord(this.name)
                }
                if(this.editMode){
                    console.log("new item is: ", newItem )
                    this.$emit('edit-item',newItem)
                }
                else{
                    //In case of a new Item the id is unique combination of name, city and state
                    newItem.id=`${newItem.name}-$${newItem.city}-${newItem.state}`.replace(/ /gi,'-')
                    this.$emit('add-item',newItem)
                }
                //clear form
                    this.id = '',
                    this.state = '',
                    this.city = '',
                    this.street = '',
                    this.name=''
            },
            capitalizeEachWord(str){
                str=str.trim()
                const words = str.split(" ");
                for (let i = 0; i < words.length; i++) {
                    words[i] = words[i][0].toUpperCase() + words[i].substr(1).trim().toLowerCase();
                }
                return words.join(" ")
            }
        }
    }
    
</script>

<style scoped>
    .create-form {
    margin-bottom: 40px;
    }
    .form-control {
    margin: 5px 0;
    }
    .form-control label {
    display: inline-block;
    width: 15%;
    font-size: 17px;
    }
    .form-control input {
    width: 80%;
    height: 40px;
    margin: 5px;
    padding: 3px 7px;
    font-size: 17px;
    }
</style>