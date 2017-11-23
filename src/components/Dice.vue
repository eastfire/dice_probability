<template>
  <div class="dice">
    <div class="number">{{number}}.</div>
      <input v-for="(face,index) in faces" class="face-input" v-model="faces[index]" type="number"></input>
    <button class="delete-dice" @click="onDelete">-</button>
  </div>
</template>

<script>
export default {
  name: 'Dice',
  props: ['number','faces'],
  data () {
    return {
      face_number: 6
    }
  },
  methods:{
    onDelete(){
      this.$emit('removeself', this.number)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.dice{
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  flex-wrap: wrap;
  flex-flow: row wrap;
}
.face-input {
  width: 25px;
}
</style>
