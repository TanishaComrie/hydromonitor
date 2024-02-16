<template>
    <VContainer fluid color= "surface">
        <VResponsive>
            <VRow max-width 1200px>
                <VCol>
                    <VSheet height= "250">
                        <p>Enter Date Range for Analysis</p>
                        <div></div>
                        <v-text-field label= "Start Date" type= "Date" v-model="start" density= "compact" variant= "solo-inverted" max-width= "300" flat></v-text-field>
                        <v-text-field label= "End Date" type= "Date" v-model="end" density= "compact" variant= "solo-inverted" max-width= "300" flat></v-text-field>
                        <VSpacer></VSpacer>
                        <VBtn @click="updateLineCharts();" text="Analyze" color="primary" variant="tonal" ></VBtn>                        
                    </VSheet>
                </VCol>
                <VCol cols= "3" align= "center">
                    <VCard title= "Temperature" width= "250" variant= "outlined" color= "primary" density= "compact" rounded= "lg">
                        <v-card-item margin-botton= "-5">
                            <v-chip-group class= "d-flex flex justify-center" color="primaryContainer" variant= "flat">
                                <v-tooltip text="Min" :location= "start">
                                    {{ temperature.min }}
                                </v-tooltip>
                                <v-tooltip text="Range" :location= "top">
                                    {{ temperature.range }}
                                </v-tooltip>
                                <v-tooltip text="Max" :location= "end">
                                    {{ temperature.max }}
                                </v-tooltip>
                            </v-chip-group>
                        </v-card-item>
                        <v-card-item margin-botton= "-5" align="center">
                            <span class="text-h1 text-primary font-weight-bold">
                                {{ temperature.avg }} 
                            </span>
                        </v-card-item>
                    </VCard>
                </VCol>
                <VCol cols= "3" align= "center">
                    <VCard title= "Humidity" width= "250" variant= "outlined" color= "primary" density= "compact" rounded= "lg">
                        <v-card-item margin-botton= "-5">
                            <v-chip-group class= "d-flex flex justify-center" color="primaryContainer" variant= "flat">
                                <v-tooltip text="Min" :location= "start">
                                    {{ humidity.min }}
                                </v-tooltip>
                                <v-tooltip text="Range" :location= "top">
                                    {{ humidity.range }}
                                </v-tooltip>
                                <v-tooltip text="Max" :location= "end">
                                    {{ humidity.max }}
                                </v-tooltip>
                            </v-chip-group>
                        </v-card-item>
                        <v-card-item margin-botton= "-5" align="center">
                            <span class="text-h1 text-primary font-weight-bold">
                                {{ humidity.avg }} 
                            </span>
                        </v-card-item>
                    </VCard>
                </VCol>
            </VRow>
            <VRow max-width 1200px>
                <VCol cols= "12">
                    <figure class= "highcharts-figure">
                        <div id= "container"></div>
                    </figure>
                </VCol>
                <VCol cols= "12">
                    <figure class= "highcharts-figure">
                        <div id= "container0"></div>
                    </figure>
                </VCol>
            </VRow>
            <VRow max-width 1200px>
                <VCol cols= "12">
                    <figure class= "highcharts-figure">
                        <div id= "container1"></div>
                    </figure>
                </VCol>
                <VCol cols= "12">
                    <figure class= "highcharts-figure">
                        <div id= "container2"></div>
                    </figure>
                </VCol>
                <VCol cols= "12">
                <figure class= "highcharts-figure">
                        <div id= "container3"></div>
                    </figure>
                </VCol>
            </VRow>
        </VResponsive>
    </VContainer>
</template>

 <script setup>
    // IMPORTS
    import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
    import { useRoute ,useRouter } from "vue-router";
    import Highcharts from "highcharts";
    import more from "highcharts/highcharts-more";
    import exporting from "highcharts/modules/exporting";
    more(Highcharts);
    exporting(Highcharts);

    import { useAppStore } from "@/store/appStore"; 
// VARIABLES
    const AppStore = useAppStore();
    const host              = ref("dbs.msjrealtms.com");
    
    
    // VARIABLES
    const router      = useRouter();
    const route       = useRoute();    
    let start         = ref("");
    let end           = ref("");
    const typing      = ref("");
    let temperature   = {"min":0,"max":0,"avg":0,"range":0};
    let humidity      = {"min":0,"max":0,"avg":0,"range":0};
    
    
    
    // COMPUTED PROPERTIES
    
    
    // WATCHERS
    


    
    // FUNCTIONS
    onMounted(()=>{
      // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
      Highcharts.chart("container0",{title:{text:"Temperature and Heat Index Analysis", align:"left"}, xAxis:{type:"datetime", title:{text:"Time"}}, yAxis:{title:{text:"Air Temperature & Heat Index"},labels:{format:"{value}°C"}},tooltip:{shared:"true", pointFormat:"Humidity: {point.x} % <br/> Temperature: {point.y} °C"}, Chart:{zoomType:"x"}});
      Highcharts.chart("container1",{title:{text:"Frequency Distribution Analysis", align:"left"}, Chart:{zoomType:"x"}});
      Highcharts.chart("container2",{title:{text:"Temperature & Heat Index Correlation Analysis", align:"left"},subtitle:{text:"Visualize the relationship between Temperature and Heat Index as well as revealing patterns or trends in the data"}, xAxis:{title:{text:"Temperature"},label:{format:"{value} °C"}}, yAxis:{title:{text:"Heat Index"},labels:{format:"{value}°C"}},tooltip:{shared:"true", pointFormat:"Temperature: {point.x} °C <br/> Heat Index: {point.y} °C"},Series:{name:"Analysis"},Chart:{zoomType:"x"}});
      Highcharts.chart("container3",{title:{text:"Humidity & Heat Index Correlation Analysis", align:"left"},subtitle:{text:"Visualize the relationship between Humidity & Heat Index as well as revealing patterns or trends in the data"}, xAxis:{title:{text:"Humidity"},label:{format:"{value} %"}}, yAxis:{title:{text:"Heat Index"},labels:{format:"{value}°C"}},tooltip:{shared:"true", pointFormat:"Humidity: {point.x} °C <br/> Heat Index: {point.y} °C"},Series:{name:"Analysis"},Chart:{zoomType:"x"}});

    });

    const updateCards = async () => {
        // Retrieve Min, Max, Avg, Spread/Range
        if(!!start.value && !!end.value){
        // 1. Convert start and end dates collected fron TextFields to 10 digit timestamps
        let startDate = new Date(start.value).getTime() / 1000;
        let endDate = new Date(end.value).getTime() / 1000;
        // 2. Fetch data from backend by calling the API functions
        const temp = await AppStore.getTemperatureMMAR(startDate,endDate);
        const humid = await AppStore.getHumidityMMAR(startDate,endDate); 
        console.log(humid) 
        temperature.max = temp[0].max.toFixed(1);        
        //3. complete for min, avg and range
        temperature.min = temp[0].min.toFixed(1);
        temperature.avg = temp[0].avg.toFixed(1);
        temperature.range = temp[0].range.toFixed(1);
        //4. complete max, min, avg and range for the humidity variable
        humidity.max = humid[0].max.toFixed(1);
        humidity.min = humid[0].min.toFixed(1);
        humidity.avg = humid[0].avg.toFixed(1);
        humidity.range = humid[0].range.toFixed(1);
        }
    }

    
    
    
    onBeforeUnmount(()=>{
      // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    });
    
    </script>
    
    <style scoped>
    /** CSS STYLE HERE */
    
    </style>
    