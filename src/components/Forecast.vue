<script setup>

import { reactive, ref, onMounted, onBeforeUpdate, onUpdated } from 'vue'

import axios from 'axios';

const props = defineProps({
    model: String,
    fcst_year: String,
    fcst_month: String,
    variable: String,
    start_lead: Number,
    end_lead: Number,
    featureID: Number,
    threshold: Number
  }
);

const state = reactive({
    'forecast':null,
    'obs_percentile':null,
    'hits_below':null,
    'misses_below':null,
    'falses_below':null,
})


onMounted(() => {
    console.log(props);
});


onUpdated(() => {

    console.log('Forecast updating '+props.leads)

    axios.get('http://localhost:5000/forecast/seasonal/'+props.model+'/'+props.fcst_year+'/'+props.fcst_month+'/'+props.variable+'/'+props.featureID+'/', 
        {
            params: {
                threshold: props.threshold,
                lead: String(props.start_lead + ',' + String(props.end_lead))
            }
        })
        .then(function (response) {
          // handle success
          console.log(response);
          state.obs_percentile = response.data.obs_percentile;
          state.hits_below = response.data.hits_below;
          state.misses_below = response.data.misses_below;
          state.falses_below = response.data.falses_below;
          state.prob_below = response.data.prob_below;
          state.fcst_below = response.data.fcst_below;
        })
      .catch(function (error) {
          // handle error
          console.log(error);
        })
      .finally(function () {
          // always executed
      });



});

</script>

<template>
    <div>
        <h3>Current Forecast</h3>
        <p>The long term probability of rainfall less that <strong>{{ threshold}} mm</strong> is <strong>{{ state.obs_percentile }}%</strong> 
            and so this has occured <strong>{{ state.hits_below + state.misses_below }} times</strong> in the past 41 years</p>
        <p>The current forecast indicates a probability of <strong> {{ state.prob_below }}%</strong>  of this occuring this year</p> 
        <p>In previous years this event was forecast <strong>{{ state.hits_below + state.falses_below }} times</strong> </p>
        <p><strong>{{ state.hits_below }} times</strong>, this forecast was correct and predicted the occurence</p>
        <p><strong>{{ state.falses_below }} times</strong>, this forecast was incorrect, the event did not occur</p>
        <p><strong>{{ state.misses_below }} times</strong>, the forecast failed to predict an actual occurence</p>
    </div>
</template>

<style scoped>
#map {
}
</style>
`