<template>
    <VContainer fluid color="surface" align="center">
        <VRow  max-width="1200px">
            <VCol cols="9">
                <figure class="highcharts-figure">
                    <div id="container"></div>

                </figure>
                
            </VCol>
            <VCol cols="3">
                <VCard  margin-bottom="5" max-width="500" color="primaryContainer" subtitle="Temperature">
                    <v-card-item> 
                        <span class="text-h3"> {{ temperature }} </span>
                    </v-card-item>
                </VCard>
                <VCard  margin-bottom="5" max-width="500" color="tertiaryContainer" subtitle="Heat Index (Feels Like)">
                    <v-card-item> 
                        <span class="text-h3"> {{ heatindex }} </span>
                    </v-card-item>
                </VCard>
                <VCard  margin-bottom="5" max-width="500" color="secondaryContainer" subtitle="Humidity">
                    <v-card-item> 
                        <span class="text-h3"> {{ humidity }} </span>
                    </v-card-item>
                </VCard>
            </VCol>
        </VRow>
        <VRow max-width="1200px" justify="start">
            <VCol cols="9">
                <figure class="highcharts-figure">
                    <div id="container1"></div>
                </figure>
                
            </VCol>
        </VRow>
    </VContainer> 
</template>

 <script setup>
    // IMPORTS
    import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
    import { useRoute ,useRouter } from "vue-router";
    import Highcharts from "highcharts";
    import more from "highcharts/highcharts-more";
    import exporting from "highcharts/modules/exporting";
    import { useAppStore } from "@/store/appStore"; 
    import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store
    import { storeToRefs } from "pinia";
    more(Highcharts);
    exporting(Highcharts);

    
  


// VARIABLES
    const AppStore = useAppStore();
    const Mqtt = useMqttStore();
    
    
    
    // VARIABLES
    const tempHiChart = ref(null);
    const router      = useRouter();
    const route       = useRoute(); 
    const points = ref(10); // Specify the quantity of points to be shown on the live graph simultaneously.
    const shift = ref(false); // Delete a point from the left side and append a new point to the right side of the graph.
    const typing      = ref("");
    const { payload, payloadTopic } = storeToRefs(Mqtt);
    const host              = ref("dbs.msjrealtms.com"); 
    let timeoutVal    = 1000;
    let nameID
    
    
    // COMPUTED PROPERTIES
    const temperature = computed(()=>{
    if(!!payload.value){
    return `${payload.value.temperature.toFixed(2)} °C`;
    }
    });

    const heatindex = computed(()=>{
    if(!!payload.value){
    return `${payload.value.heatindex.toFixed(2)} °C`;
    }
    });

    const humidity = computed(()=>{
    if(!!payload.value){
    return `${payload.value.humidity.toFixed(2)} %`;
    }
    });

    
    // WATCHERS
    watch(payload,(data)=> { 
    if(points.value > 0){ points.value -- ; }
    else{ shift.value = true; }
    tempHiChart.value.series[0].addPoint({y:parseFloat(data.temperature.toFixed(2)) ,x: data.timestamp * 1000 }, 
    true, shift.value); 
    tempHiChart.value.series[1].addPoint({y:parseFloat(data.heatindex.toFixed(2)) ,x: data.timestamp * 1000 }, 
    true, shift.value); 
    });

    watch(payload,(data)=> { 
    if(points.value > 0){ points.value -- ; }
    else{ shift.value = true; }
    tempHiChart.value.series[0].addPoint({y:parseFloat(data.humidity.toFixed(2)) ,x: data.timestamp * 1000 }, 
    true, shift.value); 
    });
    
    // FUNCTIONS
    onMounted(()=>{
      // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
      CreateCharts();

    Mqtt.connect(); // Connect to Broker located on the backend
    setTimeout( ()=>{
    // Subscribe to each topic
    Mqtt.subscribe("620156117");
    Mqtt.subscribe("620156117_pub");}, 3000);
    });
    
    
    onBeforeUnmount(()=>{
      // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    });

    const CreateCharts = async () => {
// TEMPERATURE CHART
    tempHiChart.value = Highcharts.chart('container', {
    chart: { zoomType: 'x' },
    title: { text: 'Air Temperature and Heat Index Analysis', align: 'left' },
    yAxis: { 
    title: { text: 'Air Temperature & Heat Index' , style:{color:'#000000'}},
    labels: { format: '{value} °C' } 
    },
    xAxis: {
    type: 'datetime', 
    title: { text: 'Time', style:{color:'#000000'} },
    },
    tooltip: { shared:true, },
    series: [
    {
    name: 'Temperature',
    type: 'spline',
    data: [],
    turboThreshold: 0,
    color: Highcharts.getOptions().colors[0]
    }, 
    {
    name: 'Heat Index',
    type: 'spline',
    data: [],
    turboThreshold: 0,
    color: Highcharts.getOptions().colors[1]
    } ],
    });
}
    </script>
    
    <style scoped>
    /** CSS STYLE HERE */
    
    </style>
    