<script setup>

import Plot from '../components/Plot.vue';
import Forecast from '../components/Forecast.vue';

import Map from 'ol/Map.js';
import VectorSource from 'ol/source/Vector.js';
import OSM from 'ol/source/OSM.js';
import TileLayer from 'ol/layer/Tile.js';
import VectorLayer from 'ol/layer/Vector.js'
import GeoJSON from 'ol/format/GeoJSON.js';
import View from 'ol/View.js';
import {Fill, Stroke, Style} from 'ol/style.js';

import { reactive, onMounted } from 'vue'

import axios from 'axios';




const defaultStyle = new Style({
  stroke: new Stroke({
    color: 'rgba(64, 64, 64, 0.7)',
    width: 2,
  }),
  fill: new Fill({
    color: 'rgba(0,0,0,0)'
  })
});

const hoverStyle = new Style({
  stroke: new Stroke({
    color: 'rgba(255, 128, 128, 0.7)',
    width: 4,
  }),
  fill: new Fill({
    color: 'rgba(255,64,64,0.7)'
  })
});

const selectStyle = new Style({
  stroke: new Stroke({
    color: 'rgba(255, 128, 128, 0.7)',
    width: 4,
  }),
  fill: new Fill({
    color: 'rgba(255,0,0,0.7)'
  })
});

const props = defineProps({
  features: String,
  featureID: String
});

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

const today = new Date();
const forecast_month = today.getMonth()+1;

let month_selection = Object.fromEntries(Object.entries(month_names).slice(today.getMonth(), today.getMonth()+7));

console.log('forecast month');
console.log(forecast_month);

const state = reactive({
  'hover': null, 
  'selected': null, 
  'featureID': '555',
  'active':false,
  'anomaly':null,
  'forecast_month':forecast_month,
  'months':[today.getMonth()+1,today.getMonth()+7],
  'month_selection':month_selection,
  'leads':[0,5],
  'lead_names':{
    0: 'July',
    1: 'August',
    2: 'September',
    3: 'October',
    4: 'November',
    5: 'December'
  },
  'minval': 20,
  'maxval': 400,
  'threshold':300
});


onMounted(() => {

  console.log(props);

  state.anomaly = 'none';
  state.featureID = props.featureID;

  const map = new Map({
    target: 'map',
    layers: [    
      new TileLayer({
        source: new OSM(),
      }),
      new VectorLayer({
        source: new VectorSource({
          format: new GeoJSON(),
          url: 'src/assets/data/mapping/' + props.features + '.json',
          style: defaultStyle
        }),
      }),
    ],
  
    view: new View({
      center: [2200000, -4000000],
      zoom: 7,
    }),

  });

  console.log(map);

  map.on('pointermove', function (e) {
      if (state.hover !== null && state.hover != state.selected) {
        state.hover.setStyle(defaultStyle);
        state.hover = null;
      }

      map.forEachFeatureAtPixel(e.pixel, function (f) {
        state.hover = f;
        f.setStyle(hoverStyle);
        return true;
      });

  });

  map.on('click', function (e) {

    if (state.selected !== null) {
      state.selected.setStyle(defaultStyle);
      state.selected = null;
      state.active = false;
    }

    map.forEachFeatureAtPixel(e.pixel, function (f) {
      state.selected = f;
      f.setStyle(selectStyle);

      state.active = true;
      state.featureID = f.get('id');

      return true;
    });

  });



});

function update(event) {
  console.log('BINGO');
  console.log(event);
}

</script>

<template>

    <div class="row">
      
      <div class="col-6">
        <div id="map"></div>
      </div>
      <div class="col-4">
        <h3>Seasonal cycle</h3>
        <Plot collection="observed" dataset="pr_mon_CHG-CHIRPS2.0_hexgrid_p25" variable="pr" :featureID="state.featureID" type="bar" timeagg="seasonal"></Plot>
      </div>
    </div>

    <div class="row">
      <div class="col-10">
        <h3>Select months of the year</h3>
        <v-range-slider min="8" max="14" :step="1" :ticks="state.month_selection" v-model="state.months" strict show-ticks="always"></v-range-slider>
        <v-radio-group enabled v-model="state.anomaly" inline mandatory label="Plot options">
          <v-radio key=0 label="Real values" value="none"></v-radio>
          <v-radio key=1 label="Absolute anomaly" value="absolute"></v-radio>
          <v-radio key=2 label="Relative anomaly" value="relative"></v-radio>
        </v-radio-group>
      </div>
    </div>
    <div class="row">
      <div class="col-10">        
        <Plot collection="observed" dataset="pr_mon_CHG-CHIRPS2.0_hexgrid_p25" variable="pr" :featureID="state.featureID" :startmonth="state.months[0]" :endmonth="state.months[1]" type="bar" timeagg="annual" :anomaly="state.anomaly"></Plot>
      </div>
      <div class="col-2">
        <h3>Event threshold</h3>
        <v-slider :min="state.minval" :max="state.maxval" :step="20" :draggable="false" v-model="state.threshold" thumb-label="always" direction="vertical"></v-slider>
      </div>
    </div>
    <div class="row">
      <div class="col-12">
        <Forecast model="system51" fcst_year="2023" :fcst_month="state.forecast_month" variable="pr" :start_lead="state.months[0]-state.forecast_month" :end_lead="state.months[1]-state.forecast_month" :featureID="state.featureID" :threshold="state.threshold"></Forecast>
      </div>
    </div>

</template>

<style scoped>
#map {
  height: 400px;
}
</style>
