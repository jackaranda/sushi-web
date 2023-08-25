<script setup>

import { reactive, ref, onMounted, onUpdated } from 'vue'

import axios from 'axios';

const props = defineProps({
    collection: String,
    dataset: String,
    variable: String,
    featureID: Number,
    type: String,
    timeagg: String,
    anomaly: String,
    startmonth: Number,
    endmonth: Number
  }
);

const plot = ref(null)

onMounted(() => {
    console.log(props);
});


onUpdated(() => {

    console.log('Plot updating')

    axios.get('http://localhost:5000/dataset/'+props.collection+'/'+props.dataset+'/'+props.variable+'/'+props.featureID+'/', 
        {
            params: {
                timeagg: props.timeagg,
                anomaly: props.anomaly,
                startmonth:props.startmonth, 
                endmonth:props.endmonth
            }
        })
        .then(function (response) {
          // handle success
          console.log(response);
          console.log(plot.value)

          Plotly.newPlot(plot.value,
            [{
              type: props.type,
              x: response.data.times,
              y: response.data.vals }], 
              {
                margin: { t: 0 }
              }
          );
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
  <div ref="plot"></div>
</template>

<style scoped>
#map {
}
</style>
`