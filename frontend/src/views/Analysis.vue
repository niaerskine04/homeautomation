<template>
    <v-container fluid align="center" style="color:aquamarine">
        <v-row class="pd-1" max-width="1200px" align="center" >
            <v-col class="col col1">
                <v-sheet class="pd-2" height="250">
                    <div></div>
                    <v-text-field class="mr-5" v-model="start" label="Start Date" 
                    type="date" density="compact" variant="solo-inverted" style="max-width: 300px" flat></v-text-field>
                    <v-text-field v-model="end" label="End date" type="Date" density="compact"
                     variant="solo-inverted" style="max-width: 300px" flat></v-text-field>
                    <v-spacer></v-spacer>
                    <v-btn class="text-caption" text="Analyze" @click="updateCharts(); updateCards();" color="primary" variant="tonal"></v-btn>
                </v-sheet>
            </v-col>
            <v-col class="col col2" cols="4" align="center" > 
              <v-card title="Average" width="250" outlines color="surface"  density="compact" rounded="lg" border subtitle="For the selected period">
                  <v-card-item align="center" >
                      <span class="text-h1" >
                          {{ avg.value}}
                        <span class="text-caption">GaL</span>
                      </span>
                  </v-card-item>
              </v-card>
            </v-col>
        </v-row>
        <v-row max-width="1200px" justify="start" align="center">
          <v-col class="col col1" cols="12" align = 'center'>
              <figure class="highcharts-figure">
                  <div id="container"></div>
              </figure>
          </v-col>

          <v-col class="col col2" cols="12" align="center">
              <figure class="highcharts-figure">
                  <div id="container0"></div>
              </figure>
          </v-col>
        </v-row>
    </v-container>
    
</template>

<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
import { useMqttStore } from "@/store/mqttStore"; // Import Mqtt Store
import { storeToRefs } from "pinia";
import { useAppStore } from "@/store/appStore";

import Highcharts from "highcharts";
import more from "highcharts/highcharts-more";
import Exporting from "highcharts/modules/exporting";
import { withDirectives } from "vue";
Exporting(Highcharts);
more(Highcharts);
 
// VARIABLES
const router      = useRouter();
const route       = useRoute(); 
const Mqtt = useMqttStore();
const AppStore = useAppStore();

var start = ref(null);
var end = ref(null);
const WaterMLine = ref(null); // Chart object
const HeighWLine = ref(null); // Chart object
var average= ref(null);
var avg= reactive({ value: 0 });


// FUNCTIONS
onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    Mqtt.connect(); // Connect to Broker located on the backend

    setTimeout(() => {
    // Subscribe to each topic
        Mqtt.subscribe("620155827");
        Mqtt.subscribe("620155827_pub");
    }, 3000);
    CreateCharts();
});


onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    Mqtt.unsubcribeAll();
});

const CreateCharts = async () => {
  // TEMPERATURE CHART
  WaterMLine.value = Highcharts.chart("container", {
    chart: { zoomType: "x" },
    title: { text: "Water Management Analysis", align: "left" },
    yAxis: {
      title: {
        text: "Water Reserve",
        style: { color: "#000000" },
      },
      labels: { format: "{value} Gal" },
    },

    tooltip: {
      pointFormat: "Water Reserve: {point.x} Gal ",
    },
    xAxis: {
      type: "datetime",
      title: { text: "Time", style: { color: "#000000" } },
    },
    tooltip: { shared: true },
    series: [
      {
        name: "Reserve",
        type: "line",
        data: [],
        turboThreshold: 0,
        color: Highcharts.getOptions().colors[0],
      },
    
    ],
  });


  HeighWLine.value = Highcharts.chart("container0", {
    chart: { zoomType: "x" },
    title: {
      text: "Height and Water Level Correlation Analysis",
      align: "left",
    },
    yAxis: {
      title: {
        text: "Height",
        style: { color: "#000000" },
      },
      labels: { format: "{value} in" },
    },

    xAxis: {
      title: { text: "Water Height", style: { color: "#000000" } },
      labels: { format: "{value} in" },
    },
    tooltip: {
      shared: true,
      pointFormat: "Water Height {point.x} °C ",
    },
    series: [
      {
        name: "Analysis",
        type: "scatter",
        data: [],
        turboThreshold: 0,
        color: Highcharts.getOptions().colors[0],
      },
    ],
  });

 
};
const updateCards = async () => {
  if (!!start.value && !!end.value) {

    // 1. Convert start and end dates collected fron TextFields to 10 digit timestamps
    let startDate = new Date(start.value).getTime() / 1000;
    let endDate = new Date(end.value).getTime() / 1000;
    
    // 2. Fetch data from backend by calling the API functions
    const average = await AppStore.calculate_avg_reserve(startDate, endDate);
    
    console.log(average);
    avg.value = average[0].average.toFixed(1)*10;
   
  }
};


const updateCharts = async () => {
    if (!!start.value && !!end.value) {
      // Convert output from Textfield components to 10 digit timestamps
      let startDate = new Date(start.value).getTime() / 1000;
      let endDate = new Date(end.value).getTime() / 1000;
      // Fetch data from backend
      const data = await AppStore.getAllInRange(startDate, endDate);
      // Create arrays for each plot
      let reserve = [];
      let waterheight = [];

   
      // Iterate through data variable and transform object to format recognized by highcharts
     
      data.forEach((row) => {
        reserve.push({x: row.timestamp * 1000,y: parseFloat(row.reserve.toFixed(2)),});
        waterheight.push({x: row.timestamp * 1000,y: parseFloat(row.waterheight.toFixed(2)),});
        
      });
      console.log(reserve);
      console.log(waterheight);
      // Add data to Temperature and Heat Index chart
      WaterManagLine.value.series[0].setData(reserve);
      HeighWaterLine.value.series[0].setData(waterheight);

    }
};


</script>


<style scoped>
/** CSS STYLE HERE */
.container {
  background-color: #f5f5f5;
  width: 1200px;
}

.row {
  max-width: 1200px;
}

.row1 {
  max-width: 1200px;
  padding: 1;
}

.col1 {
  border: black;
}

.sheet {
  padding: 2;
  height: 250;
}

Figure {
  border: 2px solid black
}


</style>
  