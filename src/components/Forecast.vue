<script setup>

import { reactive, ref, onMounted, onBeforeUpdate, onUpdated } from 'vue'

import axios from 'axios';

const month_names = {
  1:'Jan', 
  2:'Feb', 
  3:'Mar', 
  4:'Apr', 
  5:'May', 
  6:'Jun', 
  7:'Jul', 
  8:'Aug', 
  9:'Sep', 
  10:'Oct', 
  11:'Nov', 
  12:'Dec',
  13:'Jan',  
  14:'Feb', 
  15:'Mar', 
  16:'Apr', 
  17:'May', 
  18:'Jun'
};

const props = defineProps({
    model: String,
    fcst_year: String,
    fcst_month: Number,
    variable: String,
    start_lead: Number,
    end_lead: Number,
    featureID: String,
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

    if (state.updating == true) { 
        console.log('already updating');
        return 0; 
    }
    state.updating = true;

    console.log('Forecast updating '+props.fcst_month+' '+props.start_lead+' '+props.end_lead);

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
          //console.log(error);
        })
      .finally(function () {
          // always executed
          state.updating = false;
      });



});

</script>

<template>
    <div>
        <h3>Current Forecast</h3>
        <p>The long term probability of rainfall less that <strong>{{ threshold}} mm</strong> from <strong>{{ month_names[fcst_month + start_lead] }} to {{ month_names[fcst_month + end_lead] }}</strong> is <strong>{{ state.obs_percentile }}%</strong> 
            and this has occured <strong>{{ state.hits_below + state.misses_below }} times</strong> in the past 41 years</p>
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