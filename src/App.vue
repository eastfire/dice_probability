<template>
  <div id="app">
    <div class="add-dices">
      <button class="add-dice" @click='addD4' :disabled="dices.length>=8">+d4</button>
      <button class="add-dice" @click='addD6' :disabled="dices.length>=8">+d6</button>
      <button class="add-dice" @click='addD8' :disabled="dices.length>=8">+d8</button>
      <button class="add-dice" @click='addD10' :disabled="dices.length>=8">+d10</button>
    </div>
    <Dice v-for="dice in dices" :number="dice.number" :faces="dice.faces" v-on:removeself="onDeleteDice"/>
    <button class="start-calculate" @click='startCalculate' :disabled="dices.length<=0">Calculate</button>
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
    addD4(){
      this.dices.push({number:this.dices.length+1, faces:[1,2,3,4]})
    },
    addD6(){
      this.dices.push({number:this.dices.length+1, faces:[1,2,3,4,5,6]})
    },
    addD8(){
      this.dices.push({number:this.dices.length+1, faces:[1,2,3,4,5,6,7,8]})
    },
    addD10(){
      this.dices.push({number:this.dices.length+1, faces:[1,2,3,4,5,6,7,8,9,10]})
    },
    onDeleteDice(number){
      this.dices.splice(number-1,1);
      //recalculate number
      for ( var i = 0 ;i < this.dices.length; i++ ) {
        this.dices[i].number = i+1;
      }
    },
    startCalculate() {
      var face_indexs = [];
      var totalMap = {};
      var typeMap = {};
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
      var saveType = function(count, threshold, typeName, typeMap){
        if ( count >= threshold) {
          //save typeMap
          var key = typeName+count;
          if ( undefined === typeMap[key] ) {
            typeMap[key] = 1;
          } else typeMap[key] ++;
        }
      }

      var judgeType = function(sortedArray, typeMap) {
        var sameCount = 0;
        var straightCount = 0;
        var currentSameFace = null;
        var currentSameCount = 0;
        var currentStraightCount = 0;
        var currentStraightHead = null;
        var maxSameCount = 0;
        var maxStraightCount = 0;
        for ( var i = 0; i < sortedArray.length; i++ ) {
          var currentFace = sortedArray[i];
          if ( currentFace === currentSameFace ) {
            currentSameCount++;
          } else {
            if ( maxSameCount < currentSameCount ) {
              maxSameCount = currentSameCount;
            }
            currentSameFace = currentFace;
            currentSameCount = 1;
          }

          if ( currentFace === currentStraightHead+1 ) {
            currentStraightCount++;
          } else {
            if ( maxStraightCount < currentStraightCount ) {
              maxStraightCount = currentStraightCount;
            }

            currentStraightHead = currentFace;
            currentStraightCount = 1;
          }
        }
        if ( maxSameCount < currentSameCount ) {
          maxSameCount = currentSameCount;
        }
        if ( maxStraightCount < currentStraightCount ) {
          maxStraightCount = currentStraightCount;
        }
        if ( maxSameCount == 1 ) {
          saveType(sortedArray.length, 0, "全不同", typeMap);
        }
        saveType(maxSameCount, 2, "同数", typeMap);
        saveType(maxStraightCount, 3, "顺子", typeMap);
      }
      while (!isFinished){
        var currentFaces = [];
        var total = 0;
        for ( var i = 0; i < diceCount; i++ ){
          total+= (currentFaces[i]=this.dices[i].faces[face_indexs[i]]);
        }
        if ( undefined === totalMap[total] ) {
          totalMap[total] = 1;
        } else totalMap[total] ++;
        //judge type
        var sortedFaces = currentFaces.sort();
        judgeType(sortedFaces, typeMap)

        currentIndex = 0;
        improveIndex(this.dices);
      }

      //output
      var keys = _.keys(totalMap);
      var sum = _.reduce(this.dices, function(memo, dice){ return memo * dice.faces.length; }, 1);

      this.results = [];
      _.each ( keys, function(key) {
        this.results.push("总值"+key+"："+totalMap[key]+"/"+sum+" "+Math.round(totalMap[key]/sum*10000)/100+"%")
      },this)

      this.results.push("-----------------");

      var keys = _.keys(typeMap).sort();
      _.each ( keys, function(key) {
        this.results.push(key+"："+typeMap[key]+"/"+sum+" "+Math.round(typeMap[key]/sum*10000)/100+"%")
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
.add-dices, .start-calculate {
  width: 100%;
}
</style>
