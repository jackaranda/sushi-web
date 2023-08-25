<script setup>

import Plot from '../components/Plot.vue';

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
    color: 'rgba(128, 128, 128, 0.7)',
    width: 2,
  }),
  fill: new Fill({
    color: 'rgba(255,255,255,0.0)'
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
    color: 'rgba(255,0,0,0.9)'
  })
});

const props = defineProps({
  features: String
});

const state = reactive({
  'hover': null, 
  'selected': null, 
  'featureID': null,
  'active':false
});


onMounted(() => {

  console.log(props);

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
      center: [0, 0],
      zoom: 2,
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

</script>

<template>
  <div id="map" class="container-fluid h-100"></div>


  <div class="offcanvas offcanvas-start" :class="{ show: state.active }" tabindex="-1" id="data-panel" aria-labelledby="offcanvasLabel">
    
    <div class="offcanvas-header">
      <h5 class="offcanvas-title" id="offcanvasLabel">Visualization panel</h5>
      <button type="button" class="btn-close" data-bs-dismiss="data-panel" aria-label="Close"></button>
    </div>
    
    <div class="offcanvas-body">
      
      <ul class="nav nav-tabs" id="myTab" role="tablist">
        <li class="nav-item" role="presentation">
          <button class="nav-link active" id="home-tab" data-bs-toggle="tab" data-bs-target="#home-tab-pane" type="button" role="tab" aria-controls="home-tab-pane" aria-selected="true">Historical</button>
        </li>
        <li class="nav-item" role="presentation">
          <button class="nav-link" id="forecast-tab" data-bs-toggle="tab" data-bs-target="#forecast-tab-pane" type="button" role="tab" aria-controls="profile-tab-pane" aria-selected="false">Seasonal Forecast</button>
        </li>
      </ul>
      
      <div class="tab-content" id="myTabContent">

        <div class="tab-pane fade show active" id="home-tab-pane" role="tabpanel" aria-labelledby="home-tab" tabindex="0">
          <div class="container-md">
          <div class="row">
            <div class="col">
              Visualising for {{ features }} featureID = {{ state.featureID }}
              <Plot collection="observed" dataset="pr_mon_CHG-CHIRPS2.0_hexgrid_p25" variable="pr" :featureID="state.featureID" type="line" function="seasonal"></Plot>
            </div>
            <div class="col">
              <Plot collection="observed" dataset="pr_mon_CHG-CHIRPS2.0_hexgrid_p25" variable="pr" :featureID="state.featureID" type="bar" function="seasonal"></Plot>
            </div>
          </div>
          <div class="row">
            <Plot collection="observed" dataset="pr_mon_CHG-CHIRPS2.0_hexgrid_p25" variable="pr" :featureID="state.featureID" type="bar" function="annual"></Plot>
          </div>
        </div>

        <div class="tab-pane fade" id="forecast-tab-pane" role="tabpanel" aria-labelledby="forecast-tab" tabindex="1">
          testing
          <Plot collection="forecast" dataset="pr_mon_system5_202101-202106" variable="tprate" type="line"></Plot>
        </div>
      </div>
      </div>
    </div>
  </div>

</template>

<style scoped>
#map {
}
</style>