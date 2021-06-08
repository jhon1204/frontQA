<template>

  <div >
    <b-container>
      
      <b-row >
        <b-col md="4">
          <b v-if="time">Usando datos de :{{this.time}}</b>
          <b-card>
              
              <br/>
              <v-select v-model="poll" :value="poll" :options="pollutants" label="text" @input="updateValues"/>          
                  
                    <b-list-group flush>
                      <b-list-group-item> Punto de inicio:
                        <strong>
                          {{marker1.lat.toString()+', '+marker1.lng.toString()}}
                        </strong>
                      </b-list-group-item>
                      <b-list-group-item> Punto de inicio:
                        <strong>
                            {{marker2.lat.toString()+', '+marker2.lng.toString()}}
                        </strong>
                      </b-list-group-item>
                    </b-list-group>
                    <br/>
                    <b-card-footer>
                      <b-button variant="success" @click="getRoute">Buscar Ruta</b-button>
                    </b-card-footer>
          </b-card>
          <div id="block">  </div>
          <br/>
          <b-card no-body class="scroll">
            <b-card-body>
              <b-card-title>Instrucciones de ruta</b-card-title>
              <b-card-text>

              </b-card-text>
            </b-card-body>
            <b-list-group flush>
              <div v-if="loading">
              <b-spinner  variant="secondary"></b-spinner>
              </div>
              <div v-else>
                <b-list-group-item v-for="instruction in path.instructions" :key="instruction.interval[0]" > {{instruction.text}}</b-list-group-item>
              </div>
            </b-list-group>
          </b-card>
        </b-col>
        <b-col id="mapContainer" >
          <LMap  :zoom="zoom" :center="center" :bounds="bounds" :max-bounds="maxBounds" >
            <LTileLayer :url="url"></LTileLayer>
            <LPolyline :lat-lngs="square.latlngs" :color="square.color"></LPolyline>
            <LPolyline :lat-lngs="path.points.coordinates" color="blue"></LPolyline>
            <LMarker :lat-lng="marker1" :draggable="true" @dragend="updateMarker1"><LTooltip>Punto de Inicio</LTooltip></LMarker>
            <LMarker :lat-lng="marker2" :draggable="true" @dragend="updateMarker2"><LTooltip>Punto de Fin</LTooltip></LMarker>
            <LControlLayers
              position="topright"
              :collapsed="true"
              :sort-layers="true"
            />
            
            <LLayerGroup name="Sensores" layer-type="overlay">
             <LCircleMarker
                v-for="marker in sensors"
                :key="marker.id"
                :radius="circle.radius"
                
                :lat-lng="[marker.lat,marker.lon]"
                :color="getColor(marker.pollutantValue)"
              >
                <LPopup >{{getContent(marker.id)}}</LPopup>
             </LCircleMarker>
            </LLayerGroup>
            <LLayerGroup name="Mapa Contaminación" layer-type="overlay">
              <LPolygon v-for="cell in mapPoll"
                :key="cell.id"
                :lat-lngs="cell.geometry.coordinates[0]"
                :opacity="0.1"
                :color="getColor(cell.properties.pollution)"
              >

              </LPolygon>
                
            </LLayerGroup>
          </LMap>
          
        </b-col>
        
      </b-row>
    </b-container>
  </div>
</template>

<script>
import { LMap, LTileLayer, LMarker,LTooltip, LPolyline,LControlLayers,LPopup,LCircleMarker,LLayerGroup, LPolygon} from "vue2-leaflet";
import { latLngBounds, latLng } from "leaflet";

import sensorsData from "../../sensors.json"
import mapData from "../../densityMap.json"
import cellsData from "../../cells.json"
import pathData from "../../path.json"
import axios from "axios"
import 'vue-select/dist/vue-select.css';
export default {
  name: "Map",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LTooltip,
    LPolyline,
    LCircleMarker,
    LControlLayers,
    LPopup,
    LLayerGroup,
    LPolygon

  },
  data() {
    return {
      loading:true,
      iconSize: [32, 37],
      iconAnchor: [16, 37],
      circle: {
        radius: 5,
        color: 'red'
      },  
      path:pathData.paths[0],
      coords:[],   
      time:'',
      sensors:sensorsData.sensors,
      sensors1:[],
      mapPoll: mapData.features,
      cells: cellsData.cells,
      poll:{value: '',text:'Ninguno'},
      square: {
        latlngs: [[-12.0605,-77.0566],[-12.0350,-77.0566],[-12.0350,-77.0026],[-12.0605,-77.0026],[-12.0605,-77.0566]],
        color: 'black'
      },
      pollutants:[
        {value: 'CO',text:'CO'},
        {value: 'H2S',text:'H2S'},
        {value: 'NO2',text:'NO2'},
        {value: 'O3',text:'O3'},
        {value: 'PM10',text:'PM10'},
        {value: 'PM25',text:'PM25'},
        {value: 'SO2',text:'SO2'},
        {value: '',text:'Ninguno'},
        
        
      ],
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


    };
  },
  methods: {
    print(){
      console.log(this.poll);
    },
    getColor(value){
      let i=0
      if(this.poll.length==0){
        return 'red'
      }else{
        switch (this.poll) {
          case 'CO':
            i=value*100/10000;
            if(i>0 && i<50){
              return 'green'
            }else{
              if(i>50 && i<100){
                return 'yellow'

              }else{
                if(i>100 && i<150){
                  return 'orange'
                }else{
                  return 'red'
                }
              }
            }
          case 'H2S':
            i=value*100/150;
            if(i>0 && i<50){
              return 'green'
            }else{
              if(i>50 && i<100){
                return 'yellow'

              }else{
                if(i>100 && i<1000){
                  return 'orange'
                }else{
                  return 'red'
                }
              }
            }
          case 'NO2':
            i=value*100/200;
            if(i>0 && i<50){
              return 'green'
            }else{
              if(i>50 && i<100){
                return 'yellow'

              }else{
                if(i>100 && i<150){
                  return 'orange'
                }else{
                  return 'red'
                }
              }
            }
          case 'O3':
            i=value*100/120;
            if(i>0 && i<50){
              return 'green'
            }else{
              if(i>50 && i<100){
                return 'yellow'

              }else{
                if(i>100 && i<175){
                  return 'orange'
                }else{
                  return 'red'
                }
              }
            }
          case 'PM10':
            i=value*100/150;
            if(i>0 && i<50){
              return 'green'
            }else{
              if(i>50 && i<100){
                return 'yellow'

              }else{
                if(i>100 && i<167){
                  return 'orange'
                }else{
                  return 'red'
                }
              }
            }
          case 'PM25':
            i=value*100/25;
            if(i>0 && i<50){
              return 'green'
            }else{
              if(i>50 && i<100){
                return 'yellow'

              }else{
                if(i>100 && i<500){
                  return 'orange'
                }else{
                  return 'red'
                }
              }
            }
          case 'SO2':
            i=value*100/20;
            if(i>0 && i<50){
              return 'green'
            }else{
              if(i>50 && i<100){
                return 'yellow'

              }else{
                if(i>100 && i<625){
                  return 'orange'
                }else{
                  return 'red'
                }
              }
            }
          default:
            break;
        }
        
      }
      return 'red'

    },
    getContent(id){
      const result= this.sensors.filter(sensor => sensor.id===id)
      const result2=this.sensors1.filter(sensor => sensor.id===id)
      return result[0].id + (this.poll.value.length==0? ' \t No Seleccionó contaminante':'  \nConcentración de Contaminante :'+result2[0].pollutantValue+ ' ug/m3') 
    },

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

    },
    updateValues(){
      this.getCells()
      this.getSensors()
      this.$forceUpdate();

    },
    getCells(){
      
      if (this.poll.value.length == 0) {
        console.log('getting cells', this.poll)
        axios.get(`http://qaira-data.online/densityMap`).then((response) =>{

          this.cells=response.data.features
          this.time=response.data.timestamp
        });
        
      } else {
        console.log('getting cells',this.poll.value)
        axios.get(`http://qaira-data.online/densityMap?pollutant=${this.poll.value}`).then( (response) =>{
          this.cells=response.data.features
          this.time=response.data.timestamp 

        });
        
      }
    },
    async getSensors(){
      if (this.poll.value.length == 0) {
        console.log('getting cells', this.poll)
        axios.get(`http://qaira-data.online/sensors`).then((response) =>{
          this.sensors1=response.data.sensors
        });
        
      } else {
        console.log('getting cells',this.poll.value)
        axios.get(`http://qaira-data.online/sensors?pollutant=${this.poll.value}`).then( (response) =>{
          this.sensors1=response.data.sensors


        });
        
      }

    },
    getRoute(){
      this.loading=true
      if (this.poll.value.length == 0) {
        console.log('getting cells', this.poll)
        axios.get(`http://qaira-data.online/route?startpointlat=${this.marker1.lat}&startpointlon=${this.marker1.lng}&endpointlat=${this.marker2.lat}&endpointlon=${this.marker2.lng}`).then((response) =>{
          this.path=response.data.paths[0]
          this.loading=false
        });
        
      } else {
        console.log('getting cells',this.poll.value)
        axios.get(`http://qaira-data.online/route?pollutant=${this.poll.value}&startpointlat=${this.marker1.lat}&startpointlon=${this.marker1.lng}&endpointlat=${this.marker2.lat}&endpointlon=${this.marker2.lng}`).then( (response) =>{
          this.sensors1=response.data.paths[0]
          this.loading=false


        });
      }
      this.loading=false

    }

  },
  mounted(){
    this.getSensors();
    this.getCells();
    this.getRoute();

  },

  watch: {
    marker1: function(val){
      this.marker1=val
    },
    marker2: function(val){
      this.marker2=val
    },

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
.status-red{
  color: red;
}
.status-yellow{
  color: yellow;
}
.status-green{
  color: green;
}
.status-orange{
  color: orange;
}
.scroll {
    max-height: 200px;
    overflow-y: auto;
}

</style>