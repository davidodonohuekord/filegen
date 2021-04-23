<template>
  <div id="app">
    <div class="styled">
    <!-- shooter name -->
    <div>
    <h3>Shooter</h3>
    <input type="text" placeholder="Shooter name" v-model="shooterName"/><br>
    </div>
    <!-- options -->
    <Option v-for="(optionName, index) in Object.keys(options)" :optionName="optionName" :optionDetails="options[optionName]" :key="index" @set-all="setAll" @remove-item="removeItem" @add-item="addItem"></Option>
    <!-- <div v-for="(optionName, optionIndex) in Object.keys(options)" :key="optionIndex">
      <h3>{{optionName}}</h3>
      <span v-for="(item, itemIndex) in options[optionName]" :key="itemIndex">
        <input type="checkbox" v-model="item.value" :id="item.name" ><label :for="item.name">{{item.name}}</label>
        <button @click="removeItem(optionName, itemIndex)">Remove item</button>
        <br>
      </span>
      <input type="text" v-model="addition">
      <button @click="addItem(optionName)">Add item</button>
    </div> -->
  </div>
  <button style="margin: 0 auto; display: block;padding: 10px 15px;" @click="generateFiles" :disabled="!validOptions">Generate files</button>
  </div>
</template>

<script>

var fs = require('fs');
const electron = require('electron');
//enableRemoteModule: true
const dialog = electron.remote.dialog;
import Option from './components/Option';

// LAST_SHOTS__SENSOR_1_OPTION_5_HOUSING__147_GRAIN_990FPS__6400HZ__82MS__G17G5__DOMINIC__23_10_2020.log
  export default {
    name: 'filegen',
    components:{
      Option
    },
    data() {
      return {
        shooterName: null,
        addition: null,
        options: {
          ShotType: [
            {name: "LAST_SHOTS", value: false},
            {name: "STANDARD_SHOTS", value: false},
            {name: "HOLSTER", value: false},
            {name: "SLIDE_CATCH_RELEASE", value: false},
            {name: "RACK", value: false},
            {name: "CLEARANCE", value: false},
            {name: "RELOAD", value: false}
          ],
          Sensors: [
            {name: "SENSOR_1_OPTION_5_HOUSING", value: false},
            {name: "SENSOR_2_OPTION_5_HOUSING", value: false},
            {name: "SENSOR_3_OPTION_5_HOUSING", value: false},
            {name: "SENSOR_4_OPTION_5_HOUSING", value: false},
            {name: "SENSOR_5_OPTION_5_HOUSING", value: false},
            {name: "SENSOR_6_OPTION_5_HOUSING", value: false},
            {name: "SENSOR_7_OPTION_5_HOUSING", value: false},
            {name: "SENSOR_8_OPTION_5_HOUSING", value: false}
          ],
          Ammunition: [
            {name: "147_GRAIN_990FPS", value: false},
            {name: "17_DUMMY_ROUNDS", value: false}
          ],

          HZ: [
            {name: "6400HZ", value: true}
          ],
          Timescale: [
            {name: "82M", value: true}
          ],
          Guns: [
            {name: "G17G5", value: false},
            {name: "G19G5", value: false},
            {name: "G45G5", value: false}
          ],
        }
      }
    },
    methods: {
      setAll(name, bool) {
        for (let i = 0; i < this.options[name].length; i++) {
          this.options[name][i].value = bool;
        }
      },
      removeItem(optionName, index){
        //delete this.options[sensors]
        this.options[optionName].splice(index, 1);
        this.saveSettings();
      },
      addItem(optionName, item){
        this.options[optionName].push({
          name: item.replace(" ", "_").toUpperCase(),
          value: true
          })
        this.saveSettings();
      },
      generateFiles(){
        var date = new Date();
        var dateStamp = date.getDate() + "_" + (date.getMonth() + 1) + "_" + date.getFullYear();

        var optionKeys = Object.keys(this.options);
        var list = this.generateFileNames([""], optionKeys);
        const homeDir = require('os').homedir();
        const dir = `${homeDir}\\Desktop\\generated_files`;
        if (!fs.existsSync(dir)){
          fs.mkdirSync(dir);
        }
        for (let i = 0; i < list.length; i++){
          // append shooter name and date
          list[i] += this.shooterName.toUpperCase() + "__" + dateStamp + ".log";
          var filepath = dir + "/" + list[i];
          fs.closeSync(fs.openSync(filepath, 'w'));
        }
      },
      generateFileNames(stringArray, keysLeft){
        if (keysLeft.length == 0){
          return stringArray
        } else {
          var key = keysLeft[0];
          var rtn = [];
          for (let i = 0; i < this.options[key].length; i++){
            if (this.options[key][i].value){
              // add this value to each of the strings
              for (let j = 0; j < stringArray.length; j++){
                rtn.push(stringArray[j] + this.options[key][i].name + "__");
              }
            }
          }
          keysLeft = keysLeft.slice(1)
          return this.generateFileNames(rtn, keysLeft);
        }
      },
      saveSettings(){
        localStorage.filegenData = JSON.stringify(this.options);
      }
    },
    computed: {
      validOptions() {
        if (this.shooterName == undefined || this.shooterName == ""){
          return false;
        }
        var optionKeys = Object.keys(this.options);
        
        // for each measure
        for (let i = 0; i < optionKeys.length; i++){
          var validated = false;
          // for each element in the measure
          for (let j = 0; j < this.options[optionKeys[i]].length; j++){
            if (this.options[optionKeys[i]][j].value){
              validated = true;
            }
          }
          if (!validated){
            return false;
          }
        }
        return true;
      }
    },
    mounted(){
      if(localStorage.filegenData){
        this.options = JSON.parse(localStorage.filegenData);
      }
    }
  }
</script>

<style>

.styled{
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
  max-height: 90vh;
}
</style>
