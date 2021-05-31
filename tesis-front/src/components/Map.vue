<template>

  <div >
    <b-container>
      
      <b-row id="mapContainer">
        <b-col>
          <b-form>    
            <br/>          
                <b-form-input list="my-list-id" placeholder="Seleccionar contaminante" v-model="poll"></b-form-input>

                  <datalist id="my-list-id" >
                    <option>Más rápido</option>
                    <option v-for="poll in pollutants" v-bind:key="poll">{{ poll }}</option>
                  </datalist>
                  <br/>
                  Punto de inicio:{{marker1.lat.toString()+', '+marker1.lng.toString()}}
                  <br/>
                  <br/>
                  
                  Punto de fin: {{marker1.lat.toString()+', '+marker1.lng.toString()}}
                  <br/>
                  <br/>
                  <b-button variant="success">Buscar Ruta</b-button>
          </b-form>
          <div id="block">  </div>
          <br/>
          <h5> Instrucciones </h5>
        </b-col>
        <b-col>
          <LMap  :zoom="zoom" :center="center" :bounds="bounds" :max-bounds="maxBounds" >
            <LTileLayer :url="url"></LTileLayer>
            <LPolyline :lat-lngs="square.latlngs" :color="square.color"></LPolyline>
            <LMarker :lat-lng="marker1" :draggable="true" @dragend="updateMarker1"><LTooltip>Punto de Inicio</LTooltip></LMarker>
            <LMarker :lat-lng="marker2" :draggable="true" @dragend="updateMarker2"><LTooltip>Punto de Fin</LTooltip></LMarker>
          </LMap>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import { LMap, LTileLayer, LMarker,LTooltip, LPolyline} from "vue2-leaflet";
import { latLngBounds, latLng } from "leaflet";
import { OpenStreetMapProvider } from 'leaflet-geosearch';
export default {
  name: "Map",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LTooltip,
    LPolyline

  },
  data() {
    return {
      poll:'',
      square: {
        latlngs: [[-12.0605,-77.0566],[-12.0350,-77.0566],[-12.0350,-77.0026],[-12.0605,-77.0026],[-12.0605,-77.0566]],
        color: 'black'
      },
      pollutants:['CO','H2S','NO2','O3','PM10','PM25','SO2'],
      zoom: 13,
      center: [-12.0549,-77.0385],
      bounds: latLngBounds([
        [-12.0605, -77.0566],
        [-12.0350, -77.0026]
      ]),
      maxBounds: latLngBounds([
        [-12.0605, -77.0566],
        [-12.0350, -77.0026]
      ]),
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      marker1: latLng(-12.0549,-77.0385),
      marker2:latLng(-12.0571,-77.0266),
      geosearchOptions: { // Important part Here
        provider: new OpenStreetMapProvider(),
      }

    };
  },
  methods: {

    updateMarker1(event){
      this.marker1=event.target._latlng
    },
    updateMarker2(event){
      this.marker2=event.target._latlng
    },
    addMarker(event){
      this.markers.push(event.latLng)
    },
    showMarker(index){
      console.log(this.markers[index])

    }
  },
  watch: {
    marker1: function(val){
      this.marker1=val
    },
    marker2: function(val){
      this.marker2=val
    }
  }
}
</script>

<style scoped>
#mapContainer {
 width: 80vw;
 height: 90vh;
}
.block{
  margin: 10px;
}
</style>