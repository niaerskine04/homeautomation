<template>
    <v-container class="container" fluid align="center" justify="center">
        <v-row class="bg-surface">
            <v-col class="col col2" align="center">
                <v-sheet>
                    <v-card class="text-secondary" title="Combination" color="surface" subtitle="Enter your four-digit passcode" variant="tonal" flat>
                    </v-card>
                </v-sheet>
                <v-otp-input length="4" style="color:blueviolet">
                </v-otp-input>
            </v-col>
        </v-row>
        <v-row>
            <v-col class="col col2" align="center">
                <v-btn class="rounded-t-lg" align="center" variant="tonal" style="color:aqua" @click="readCode()" >
                    Submit
                </v-btn>
            </v-col>
        </v-row>
    </v-container>
</template>

<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store 
import { storeToRefs } from "pinia";
 
 
 
// VARIABLES
const router      = useRouter();
const route       = useRoute();
const Mqtt        = useMqttStore(); 
const { payload, payloadTopic } = storeToRefs(Mqtt);  


// FUNCTIONS
onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    Mqtt.connect(); // Connect to Broker located on the backend

    setTimeout(() => {
    // Subscribe to each topic
        Mqtt.subscribe("620155827");
        Mqtt.subscribe("620155827_pub");
    }, 3000);
});


onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    Mqtt.unsubcribeAll();
});

const updateGauge = async () => {
  if (!!start.value && !!end.value) {
    // Convert output from Textfield components to 10 digit timestamps
    let startDate = new Date(start.value).getTime() / 1000;
    let endDate = new Date(end.value).getTime() / 1000;
    // Fetch data from backend
    const data = await AppStore.getAllInRange(startDate, endDate);
    // Create arrays for each plot
    let gaugepoint = [];
    
    // Iterate through data variable and transform object to format recognized by highcharts
    data.forEach((row) => {
      gaugepoint.push({
        x: parseFloat(row.radar.toFixed(2)),
        y: parseFloat(row.waterheight.toFixed(2)),
      }
    )
    });
     
    // Add data to water analysis chart
    heighwaterLine.series[0].setData(scatterPoints1);
  }
};

const readCode = async () => {
  const foo = await AppStore.getSetCombination(passcode.value);
};

</script>


<style scoped>
/** CSS STYLE HERE */
.container {
  background-color: black;
  width: 1200px;
}


</style>
  