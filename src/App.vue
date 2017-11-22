<template>
  <div id="app">
    <button class="add-dice" @click='addDice' :disabled="dices.length>=8">+</button>
    <Dice v-for="dice in dices" :number="dice.number" :faces="dice.faces"/>
    <button class="start-calculate" @click='startCalculate'>Calculate</button>
    <div v-for="result in results">{{result}}</div>
  </div>
</template>

<script>
import Dice from './components/Dice'

export default {
  name: 'app',
  data () {
    return {
      dices: [{number:1, faces:[1,2,3,4,5,6]},{number:2, faces:[1,2,3,4,5,6]}],
      results: []
    }
  },
  components: {
    Dice
  },
  methods: {
    addDice(){
      this.dices.push({number:this.dices.length+1, faces:[1,2,3,4,5,6]})
    },
    startCalculate() {
      var face_indexs = [];
      var totalMap = {};
      var diceCount = this.dices.length
      var currentIndex = 0;
      for ( var i = 0 ; i < diceCount; i++ ) {
        face_indexs.push(0);
        for ( var j = 0; j < this.dices[i].faces.length; j++ ) {
          this.dices[i].faces[j] = parseInt(this.dices[i].faces[j]);
        }
      }
      var isFinished = false;
      var improveIndex = function (dices){
        face_indexs[currentIndex]++;
        if ( face_indexs[currentIndex] >= dices[currentIndex].faces.length ) {
          face_indexs[currentIndex] = 0;
          currentIndex++;
          if ( currentIndex >= diceCount ) {
            isFinished = true;
          } else {
            improveIndex(dices);
          }
        }
      }
      while (!isFinished){

        var total = 0;
        for ( var i = 0; i < diceCount; i++ ){
          total+= this.dices[i].faces[face_indexs[i]];
        }
        if ( undefined === totalMap[total] ) {
          totalMap[total] = 1;
        } else totalMap[total] ++;

        currentIndex = 0;
        improveIndex(this.dices);
      }

      //output
      var keys = _.keys(totalMap);
      var sum = _.reduce(keys, function(memo, key){ return memo + totalMap[key]; }, 0);

      this.results = [];
      _.each ( keys, function(key) {
        this.results.push("总值"+key+"："+totalMap[key]+"/"+sum+" "+Math.round(totalMap[key]/sum*10000)/100+"%")
      },this)
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.add-dice, .start-calculate {
  width: 100%;
}
</style>
