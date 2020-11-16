<template>
    <div>
      <h3>{{optionName}}</h3>
      <button @click="checkAll">Check all</button>
      <button @click="uncheckAll">Uncheck all</button><br>
      <span v-for="(item, itemIndex) in optionDetails" :key="itemIndex">
        <input type="checkbox" v-model="item.value" :id="item.name" ><label :for="item.name">{{item.name}}</label>
        <button @click="$emit('remove-item', optionName, itemIndex)">Remove item</button>
        <br>
      </span>
      <input type="text" v-model="addition">
      <button @click="addItem">Add item</button>
  </div>
</template>

<script>

  export default {
      props:['optionName', 'optionDetails'],
      data(){
          return{
              addition: null
          }
      },
      methods: {
          addItem(){
              this.$emit('add-item', this.optionName, this.addition);
              this.addition = null
          },
          uncheckAll() {
            this.$emit('set-all', this.optionName, false);
          },
          checkAll() {
            this.$emit('set-all', this.optionName, true);
          }
      }
  }
</script>