<template>
  <div class="flex-gap">
    <v-btn v-if="storeStep.step > -1" @click="storeStep.step = storeStep.step - 1">
      Previous
    </v-btn>
    <v-btn v-if="storeStep.step < 11" @click="storeStep.step = storeStep.step + 1"> Next </v-btn>
        <v-snackbar
      v-model="snackbar"
      :color="message === 'Success' ? '#4fbba9' : 'red'"
      timeout="2000"
    >
      {{ message }}

      <template v-slot:actions>
        <v-btn
          color="white"
          variant="text"
          @click="snackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>

<script>
import { useTcpStore, useStep } from '@/stores/counter'

export default {
  name: 'Setup',
  data() {
    return {
      storeStep: useStep(),
      storeTCP: useTcpStore(),
      snackbar:false,
      message:""
    }
  },
    watch: {
    storeStep:{
      async handler() {
        try{
          this.snackbar = false
          this.message = ""
          
        const data = {
          ...this.storeTCP.web,
          ...this.storeTCP.mqtt,
          ...this.storeTCP.inverter1,
          ...this.storeTCP.inverter2,
          ...this.storeTCP.inverter3,
          ...this.storeTCP.influx,
          ...this.storeTCP.homeAssistant,
          ...this.storeTCP.tariffs,
          ...this.storeTCP.miscellaneous,
          ...this.storeTCP.palm,
          ...this.storeTCP.evc
        }
        var host = window.location.protocol + "//" + window.location.hostname+":6345/settings"
        //var host = "http://127.0.0.1:6345/settings"
        const setResponse = await fetch(host,{
        method:"POST",
        headers:{
          "Content-Type":"application/json"
        },
        body:JSON.stringify(data)
      })
      if(setResponse.ok){
       this.snackbar = true
       this.message = `Success`
       var host = window.location.protocol + "//" + window.location.hostname+":6345/settings"
       const getResponse = await fetch(host)

        const getJSON = {...await getResponse.json()}

        if(!getResponse.ok){
          this.snackbar = true
          this.message = `Server Error: ${setResponse.statusText}`
        }

        Object.keys(data).map((key)=>{
          if(key in this.storeTCP.web){
            this.storeTCP.web[key] = getJSON[key]
          }else if(key in this.storeTCP.mqtt){
            this.storeTCP.mqtt[key] = getJSON[key]
          }else if(key in this.storeTCP.inverter1){
            this.storeTCP.inverter1[key] = getJSON[key]
          }else if(key in this.storeTCP.inverter2){
            this.storeTCP.inverter2[key] = getJSON[key]
          }else if(key in this.storeTCP.inverter3){
            this.storeTCP.inverter3[key] = getJSON[key]
          }else if(key in this.storeTCP.influx){
            this.storeTCP.influx[key] = getJSON[key]
          }else if(key in this.storeTCP.homeAssistant){
            this.storeTCP.homeAssistant[key] = getJSON[key]
          }else if(key in this.storeTCP.tariffs){
            this.storeTCP.tariffs[key] = getJSON[key]
          }else if(key in this.storeTCP.miscellaneous){
            this.storeTCP.miscellaneous[key] = getJSON[key]
          }else if(key in this.storeTCP.palm){
            this.storeTCP.palm[key] = getJSON[key]
          }else if(key in this.storeTCP.evc){
            this.storeTCP.evc[key] = getJSON[key]
          }else {
            return
          }
        })
      }else{
        this.snackbar = true
        this.message = `Server Error: ${setResponse.statusText}`
      }
        }catch(e){
          this.snackbar = true
          this.message = `Error: ${e}`
        }
    },
    deep:true
    }
  },
}
</script>

<style scoped>
.flex-gap{
  display: inline-flex;
  flex-wrap: wrap;
  gap: 12px;
}
</style>