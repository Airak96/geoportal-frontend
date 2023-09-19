<script setup>
  import MenuItem from '../components/MenuItem.vue'
  import MapItem from '../components/MapItem.vue'
  import InfoModal from '../components/InfoModal.vue';
  import { transform } from 'ol/proj'
  import { ref, inject } from 'vue';

  // store imports
  import { useCategoriesStore } from '../stores/categories.store'

  const categoriesStore = useCategoriesStore();
  const data = ref([]);
  const selected = ref([]);

  categoriesStore.getAllData()
    .then(res => {
      let arr = res.data;
      data.value = arr.map(obj => ({
        ...obj,
        layers: obj.layers.map(lay => ({ ...lay, show: false })),
      }));
    })
    .catch(error =>console.log(error));

  // formato de fuentes
  const childComponentRef = ref(null);
  const map = ref(null);

  // config mapa
  const long = -79.05054214735792;
  const lati = -4.887745621654;
  const center = ref(transform([long, lati], 'EPSG:4326', 'EPSG:3857'));
  const projection = ref("EPSG:3857");
  const zoom = ref(9);
  const rotation = ref(0);

  const format = inject("ol-format");
  const GeoJSON = new format.WFS();

  //                 oeste           sur           este          norte
  // const extent = ref([-8859051.0774, -670624.0975, -8642349.8685,-473395.3977]);

  // sistema de referencia coordenadas
  const projectionName = "EPSG:32717";
  const projectionDef = "+proj=utm +zone=17 +south +datum=WGS84 +units=m +no_defs +type=crs";

  const clicked = (event) => {
    // console.log(event);
    map.value.forEachFeatureAtPixel([event.x, event.y], function(feature, layer) {
      if(feature && feature.values_){
        console.log(feature.values_)
        childComponentRef.value?.openModal(feature.values_)
      }
    });
  };

  const urlFunction = (extent, resolution, projection) => {
    const url =
      "http://localhost:8080/geoserver/Ecuador/wfs?service=WFS&" +
      "version=1.1.0&request=GetFeature&typename=shp2";
    return url;
  };
</script>
<template>
  <div class="grid grid-cols-5 gap-3 h-full">
    <div class="col-span-5 relative h-full">      
      <!-- filtro del mapa -->
      <div class="absolute top-10 right-10 z-10">
        <div class="bg-white rounded-lg px-5 py-5 shadow-md mb-5">
          <form>
            <fieldset>
              <legend class="w-full px-2">
                <button type="button" class="flex w-full items-center justify-between px-2 text-gray-400 hover:text-gray-500" aria-controls="filter-section-0" aria-expanded="false">
                  <span class="text-lg font-medium text-gray-900">Filtro de Capas</span>
                </button>
              </legend>
              <MenuItem 
                v-if="data?.length && data.length > 0"
                v-for="item in data"
                :item="item"
                :lvl="1"
                :legends="selected"
              />
            </fieldset>
          </form>
        </div>

        <div class="bg-white rounded-lg px-5 py-5 shadow-md">
          <legend class="w-full px-2">
            <button type="button" class="flex w-full items-center justify-between px-2 text-gray-400 hover:text-gray-500" aria-controls="filter-section-0" aria-expanded="false">
              <span class="text-lg font-medium text-gray-900">Leyenda</span>
            </button>
          </legend>
          <template v-if="selected?.length && selected.length > 0">
            <template v-for="obj in selected">
              <div class="flex items-center gap-3 pl-4 mt-3" v-for="legend in obj.legends">
                <div class="w-[20px] h-4" :style="{ backgroundColor: legend.color }"></div>
                <div class="text-sm text-gray-500">{{ legend.description }}</div>
              </div>              
            </template>             
          </template>

          <template v-if="selected?.length === 0">
            <div class="pl-4 text-sm italic text-gray-500 mt-1">
              (No hay capas seleccionadas)
            </div>
          </template>
        </div>
      </div>

      <!-- MAPA (Open Layers: vue3-openlayers) -->
      <div class="rounded-xl overflow-hidden h-full">
        <ol-map
          ref="map"
          :loadTilesWhileAnimating="true"
          :loadTilesWhileInteracting="true"
          style="height: 100%"
          @click="clicked"
        >
          <!-- registro de proyeccion -->
          <ol-projection-register
            :projectionName="projectionName"
            :projectionDef="projectionDef"
          />
          <!-- FIN registro de proyeccion -->

          <!-- configuracion del mapa -->
          
          <!-- :center     = coordenadas para posicionar el centro del mapa -->
          <!-- :rotation   = nivel de rotacion del mapa -->
          <!-- :zoom       = aumento predeterminado -->
          <!-- :projection = sistema de referencia de coordenadas -->
          <ol-view
            ref="view"
            :center="center" 
            :rotation="rotation"
            :zoom="zoom"
            :projection="projection"
          />
          <!-- FIN configuracion del mapa -->

          <ol-webgl-tile-layer :zIndex="1">
            <ol-source-osm />
          </ol-webgl-tile-layer>


          <!-- PRUEBA DE CAPAS GEOSERVER -->
          <ol-tile-layer :zIndex="1">
            <ol-source-tile-wms
              url="http://localhost:8080/geoserver/wms"
              layers="Ecuador:shp2"
              serverType="geoserver"
              :transition="0"
            />
          </ol-tile-layer>

          <!-- <ol-tile-layer :zIndex="1">
            <ol-source-tile-wms
              url="http://localhost:8080/geoserver/wms"
              layers="Ecuador:peru_distritos"
              serverType="geoserver"
              :transition="0"
            />
          </ol-tile-layer> -->

          <!-- <ol-vector-layer>
            <ol-source-vector
              :url="urlFunction"
              :format="GeoJSON"
              :projection="projection"
            />
          </ol-vector-layer> -->

          <!-- zoom del mapa -->
          <ol-zoom-control />
          <!-- FIN zoom del mapa -->

          <!-- barra de zoom del mapa -->
          <ol-zoomslider-control />
          <!-- FIN barra de zoom del mapa -->

          <!-- escala del mapa -->
          <ol-scaleline-control />
          <!-- FIN escala del mapa -->

          <!-- capa de vector -->
          <template v-for="category in data">
            <MapItem 
              :item="category"
              :lvl="1"
            />
          </template>
        </ol-map>
        <!-- FIN mapa -->
      </div>
    </div>
  </div>
  <InfoModal ref="childComponentRef" />
</template>