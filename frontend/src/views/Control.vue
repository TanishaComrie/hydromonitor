<template>
     <VContainer fluid align="center" justify="center">
        <VResponsive>
        <VRow max-width="1200">
         <VCol align= "center" justify="center">
            <VSheet class= "rounded-t-lg mb-1" color= "surface" elevation= "0" max-width= "800" width= "100%" flat>
            <VCard title= "LED Controls" subtitle= "Recent settings" variant= "tonal" color= "surface"></VCard>
            </VSheet>
            <VSheet class= "mb-1" color= "surface" elevation= "0" max-width= "800" width= "100%">
                <VCard class= "surface" variant= "tonal">
                    <VSlider class= "pt-2 bg-surface" v-model= "led.brightness" append-icon= "mdi:mdi-car-light-high" density= "compact" thumb-size= "16" color= "secondary" label= "Brightness" direction= "horizontal" min= "0" max= "250" step= "10" show-tick thumb-label= "always"></VSlider>
                </VCard>
            </VSheet>
            <VSheet class= "mb-1" justify= "center" align= "center" color= "surface" elevation= "0" max-width= "800" width= "100%">
                <VCard class= "pt-5" color= "surface" variant= "tonal" >
                    <VSlider class= "pt-2 bg-surface" v-model= "led.nodes" append-icon= "mdi:mdi-led-on" density= "compact" thumb-size= "16" color= "secondary" label= "LED Nodes" direction= "horizontal" min= "1" max= "7" step= "1" show-ticks thumb-label= "always"></VSlider>
                </VCard>
            </VSheet>
            <VSheet class= "mb-1 pa-2" justify= "center" align= "center" color= "surface" elevation= "0" max-width= "800" width= "100%">
                <VProgressCircular rotate= "0" size= "200" width= "15" :color="indicatorColor"></VProgressCircular>
            </VSheet>
         </VCol>
         <VCol align= "center" justify="center">
            <VColorPicker v-model= "led.color" :model-value="led.nodes *15">
                <span class="text-onSurface font-weight-bold">{{ led.nodes }} LED(s)</span>
            </VColorPicker>
         </VCol>
        </VRow>
        </VResponsive>
    </VContainer>
</template>

 <script setup>
    // IMPORTS
    import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
    import { useRoute ,useRouter } from "vue-router";
    
    
    
    // VARIABLES
    const led         = reactive ({"brightness":255, "nodes":1, "color":{r:255, b:255, g:0, a:1}});
    const router      = useRouter();
    const route       = useRoute(); 
    let timer, id     = 1000;
    
    
    // COMPUTED PROPERTIES
    const indicatorColor = computed(()=>{
        return `rgba(${led.color.r},${led.color.g},${led.color.b},${led.color.a})`
    })
    
    // WATCHERS 
    watch(led,(controls)=>{
        clearTimeout(ID);
        ID = setTimeout(()=>{ 
    const message =
    JSON.stringify({"type":"controls","brightness":controls.brightness,"leds":controls.nodes,"color": controls.color});
    Mqtt.publish("620156117_sub",message); // Publish to a topic subscribed to by the hardware 
    },1000) 
    })

    
    // FUNCTIONS
    onMounted(()=>{
      // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    });
    
    
    onBeforeUnmount(()=>{
      // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    });
    
    </script>
    
    <style scoped>
    /** CSS STYLE HERE */
    
    </style>
    
  