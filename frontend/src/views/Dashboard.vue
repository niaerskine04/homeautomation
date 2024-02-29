<template>
    <v-container>
        <v-row class="row row1" max-width="1200px" justify="start" align="center">
            <v-col class="col col1" cols="2"  align="left" >
                <v-sheet>
                    <v-slider class="pt-2 bg-surface" v-model="radarValue"
                    density="compact" thumb-label width="100px" color="green"
                    label="Height(in)" direction="vertical" track-size="50" showTicks  ></v-slider>
                </v-sheet>
            </v-col>
            <v-col class="col col2" cols="9" align="left">
                <v-sheet border max="120">
                    <figure class="highcharts-figure">
                        <div id="container"></div>
                    </figure>
                </v-sheet>
            </v-col>
        </v-row>
        <v-row class="row row2">
            <v-col class="col col1" cols="6" md="8" align="left">
                <v-card class="text-secondary bg-background"  color="surface" outlined>
                    <figure class="highcharts-figure">
                        <div id="container1"></div>
                    </figure>
                </v-card>
            </v-col>
            <v-col class="col col2" cols="3">
                <v-card class="text-secondary" title="Water Level" color="surface" subtitle="Capacity Remaining" variant="tonal" flat>
                    <div id="fluid-meter"></div>         
                </v-card>  
            </v-col>
        </v-row>
        <v-dialog v-model="overflowDialog" max-width="400">
            <v-card title="Overflow Detected">
                <v-btn color="primary" @click="overflowDialog = false">Close</v-btn>
            </v-card>
        </v-dialog>
    </v-container>
</template>

<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
import Highcharts from "highcharts";
import more from "highcharts/highcharts-more";
import Exporting from "highcharts/modules/exporting";
import { withDirectives } from "vue";
Exporting(Highcharts);
more(Highcharts);

import { useMqttStore } from "@/store/mqttStore"; // Import Mqtt Store
import { storeToRefs } from "pinia";
import { useAppStore } from "@/store/appStore";

 
 
// VARIABLES
const router      = useRouter();
const route       = useRoute();  
const Mqtt = useMqttStore();
const AppStore = useAppStore();
const areaGraph = ref(null); // Chart object
const waterReserveGraph = ref(null); // Chart object
const radarValue = ref(50);
const point= ref(10);
const shift= ref(false);
var fm = new FluidMeter();

const overflowDialog = ref(false);

const waterheight= computed(()=>{
    if(!!payload.value){
      return '${payload.value.waterheight.toFixed(2)} inches';
    }
    }
  );

  const reserves= computed(()=>{
    if(!!payload.value){
      return '${payload.value.reserves.toFixed(2)} gallons';
    }
    }
  );

  const percentage= computed(()=>{
    if(!!payload.value){
      return '${payload.value.percentage.toFixed(2)}';
    }
    }
  );


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

function data (){
      return {radarValue: 50}
}

const CreateCharts = async () => {
    areaGraph.value = Highcharts.chart("container", {
        chart: { zoomType: "x" },
        title: { text: "Water Reserve (10 min)", align: "left",},

        yAxis: {
        title: {
            text: "Water Level",
            style: { color: "#000000" },
        },
        labels: { format: "{value} Gal" },
        },

        xAxis: {
        type: "datetime",
        title: { text: "Time", style: { color: "#000000" } },
        },
        tooltip: { shared: true },
        series: [
        {
            name: "Water",
            type: "area",
            data: [],
            turboThreshold: 0,
            color: Highcharts.getOptions().colors[0],
        },
        ],
    });
    waterReserveGraph.value = Highcharts.chart("container1", {
        chart: { zoomType: "x" },
        title: { text: "Water Reserves", align: "left",},

    yAxis: {
            min: 0,
            max: 1000,
            tickPixelInterval: 72,
            tickPosition: 'inside',
            tickColor: Highcharts.defaultOptions.chart.backgroundColor || '#FFFFFF',
            tickLength: 20,
            tickWidth: 2,
            minorTickInterval: null,
            labels: { distance: 20, style: { fontSize: '14px' } },
            lineWidth: 0,
            plotBands: [{ from: 0, to: 200, color: '#DF5353', thickness: 20 }, { from: 200, to: 600, color: '#DDDF0D', thickness: 20
        }, { from: 600, to: 1000, color: '#55BF3B', thickness: 20 }]
            },

            tooltip: { shared:true, },
            pane: { startAngle: -90, endAngle: 89.9, background: null, center: ['50%', '75%'], size: '110%' },
        
            series: [{
            type:'gauge',
            name: 'Water Capacity',
            data:[0],
            tooltip: { valueSuffix: ' Gal' },
            dataLabels: { format: '{y} Gal', borderWidth: 0, color: ( Highcharts.defaultOptions.title &&
            Highcharts.defaultOptions.title.style && Highcharts.defaultOptions.title.style.color ) || '#333333', style: { fontSize: '16px' }
    },
            dial: { radius: '80%', backgroundColor: 'gray', baseWidth: 12, baseLength: '0%', rearLength: '0%' },
            pivot: { backgroundColor: 'gray', radius: 6 }
            }]
            
        });

        fm.init({
    targetContainer: document.getElementById("fluid-meter"),
    fillPercentage: 15,
    options: {
        fontSize: "70px",
        fontFamily: "Arial",
        fontFillStyle: "white",
        drawShadow: true,
        drawText: true,
        drawPercentageSign: true,
        drawBubbles: true,
        size: 300,
        borderWidth: 25,
        backgroundColor: "#e2e2e2",
        foregroundColor: "#fafafa",
        foregroundFluidLayer: {
        fillStyle: "purple",
        angularSpeed: 100,
        maxAmplitude: 12,
        frequency: 30,
        horizontalSpeed: -150
        },
        backgroundFluidLayer: {
        fillStyle: "pink",
        angularSpeed: 100,
        maxAmplitude: 9,
        frequency: 30,
        horizontalSpeed: 150
        }
    }
    });
}


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
     
    // Add data to Temperature and Heat Index chart
    WaterCapacityChart.series[0].setData(gaugepoint);
  }
};

watch(() => percentage, (newValue) => {
  // Check if the percentage exceeds 100
  if (newValue > 100) {
    // If it exceeds 100, set overflowDialog to true to display the dialog
    overflowDialog.value = true;
  }
});


</script>


<style scoped>
/** CSS STYLE HERE */
.chart{
    border:black;
}

</style>
  