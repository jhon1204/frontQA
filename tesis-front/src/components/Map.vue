<template>

  <div >
    <b-container>
      
      <b-row >
        <b-col md="4">
          <b v-if="time">Usando datos de :{{this.time.substring(4,this.time.length)}}</b>
          <b-card>
              
              <br/>
              <v-combobox v-model="poll" :items="pollutants" label="text" @input="updateValues"/>          
                  
                    <b-list-group flush>
                      <b-list-group-item> Punto de inicio:
                        <strong>
                          {{ marker1.lat? (marker1.lat +', '+ marker1.lng):(marker1[1].toFixed(5)+','+marker1[0].toFixed(5)) }}
                        </strong>
                      </b-list-group-item>
                      <b-list-group-item> Punto de inicio:
                        <strong>
                            {{ marker2.lat? (marker2.lat +', '+ marker2.lng):(marker2[1].toFixed(5)+','+marker2[0].toFixed(5)) }}
                        </strong>
                      </b-list-group-item>
                    </b-list-group>
                    <br/>
                    <b-card-footer>
                      <b-container>
                        <b-row>
                        <b-col>
                          <b-button variant="success" @click="getRoute">Buscar Ruta</b-button>
                        </b-col>
                        <b-col>
                          <div v-if="!updated" style="font-weight:bold;color:red"> Falta actualizar ruta</div>
                        </b-col></b-row>
                      </b-container>
                    </b-card-footer>
          </b-card>
          <div id="block">  </div>
          <br/>
          <b-card no-body class="scroll">
            <b-card-body>
              <b-card-title>Instrucciones de ruta </b-card-title>
              <b-card-text >
                <div v-if="accumulated && !(poll==='Ninguno') && !loading">
                {{'Contaminación acumulada de ruta: '+accumulated.toFixed(2)+' ug/m2'}}
                </div>
              </b-card-text>
            </b-card-body>
            <b-list-group flush>
              <div v-if="loading">
              <b-spinner  variant="secondary"></b-spinner>
              </div>
              <div v-else >
                <b-list-group-item  v-for="instruction in instructions" :key="instruction.interval[0]" > {{instruction.text}}</b-list-group-item>
              </div>
            </b-list-group>
          </b-card>
        </b-col>
        <b-col id="mapContainer" >
          <MglMap :accessToken="accessToken" :mapStyle="mapStyle" :center="center" :maxBounds="bounds" >
            <MglMarker :coordinates="marker1" :draggable="true" @dragend="updateMarker1" >
              <MglPopup :showed="true" anchor="top">
                Punto de Inicio
              </MglPopup>
              
            </MglMarker>
            <MglMarker :coordinates="marker2" :draggable="true" @dragend="updateMarker2" anchor="top" >
              <MglPopup :showed="true">
                Punto de Fin
              </MglPopup>
            </MglMarker>
            
            <MglMarker v-for="sensor in sensors" :key="sensor.id" :coordinates="[sensor.lon,sensor.lat]">
              <v-icon slot="marker" large style="position:absolute" color="red" >mdi-crosshairs-gps</v-icon>
              <MglPopup >
                <div>
                  <div>
                    {{'Sensor '+sensor.id}}
                  </div>
                  <div>
                    {{poll==="Ninguno"?"No se seleccionó contaminante":("   Concentración de contaminante: "+sensor.pollutantValue+' ug/m3')}}
                  </div>
                </div>
              </MglPopup>
            </MglMarker>
            <MglGeojsonLayer type="fill" :source="mapPoll" sourceId="mapPoll" layerId="pollution-layer" :layer="pollutionLayer"></MglGeojsonLayer>
            
            <MglGeojsonLayer type="line" :source="path" sourceId="path" layerId="line-layer" :layer="lineLayer"></MglGeojsonLayer>

          </MglMap>
          
        </b-col>
        
      </b-row>
    </b-container>
  </div>
</template>

<script>
import MapBox from "mapbox-gl";
import {MglMap, MglMarker,MglPopup, MglGeojsonLayer} from "vue-mapbox"
import sensorsData from "../../sensors.json"
import mapData from "../../densityMap.json"
import cellsData from "../../cells.json"
import pathData from "../../path.json"
import axios from "axios"
import 'vue-select/dist/vue-select.css';
export default {
  name: "Map",
  components: {
    /* LMap,
    LTileLayer,
    LMarker,
    LTooltip,
    LPolyline,
    LCircleMarker,
    LControlLayers,
    LPopup,
    LLayerGroup,
    LPolygon */
    MglMap,
    MglMarker,
    MglPopup,
    MglGeojsonLayer


  },
  created(){
    this.mapbox=MapBox;
    
  },
  data() {
    return {
      updated:true,
      accumulated:pathData.paths[0].weight,
      pollutionLayer:{id:'pollution-layer',source:'mapPoll',type: "fill",paint:{ "fill-color":{type: 'identity',property: 'color'},"fill-opacity":0.1}},
      lineLayer:{id:'line-layer',source:'path',type: "line",paint:{ "line-color":"green","line-width":3}},
      accessToken:'pk.eyJ1IjoiamhvbmJlbml0ZXMiLCJhIjoiY2twY3o2a25qMWF0cDJybGFxOW0ydXd5ZyJ9.tL58AV2mr3WkZyyQWaeMZw',
      mapStyle:'mapbox://styles/mapbox/streets-v11',
      loading:false,
      iconSize: [32, 37],
      iconAnchor: [16, 37],
      circle: {
        radius: 5,
        color: 'red'
      },
      instructions:pathData.paths[0].instructions,  
      path:{data:{type:'FeatureCollection',features:[{geometry:pathData.paths[0].points, type:"Feature"}]}}, 
      time:'',
      
      sensors:sensorsData.sensors,
      mapPoll: {data:mapData},
      cells: cellsData.cells,
      poll:'Ninguno',
      pollutants:[
              'CO',
              'H2S',
              'NO2',
              'O3',
              'PM10',
              'PM25',
              'SO2',
              'Ninguno'
        
        
      ],
      zoom: 13,
      center: [-77.0385,-12.0549],
       bounds: [
        [ -77.0566,-12.0605],
        [ -77.0026,-12.0350]
      ],
      marker1: [-77.0385,-12.0549],//latLng(-12.0549,-77.0385),
      marker2:[-77.0266,-12.0571]//latLng(-12.0571,-77.0266),


    };
  },
  methods: {
    print(){
      console.log(this.poll);
    },
    print2(event){
      console.log(event)
    },
    getColor(value){

      let i=0
      if(this.poll==='Ninguno'){
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

    updateMarker1(event){
      this.marker1=[event.marker._lngLat.lng,event.marker._lngLat.lat]
    },
    updateMarker2(event){
      this.marker2=[event.marker._lngLat.lng,event.marker._lngLat.lat]
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
      this.updated=false

    },
    async getCells(){
      
      if (this.poll === 'Ninguno') {
        console.log('getting cells', this.poll)
        var response=await axios.get(`http://qaira-data.online:8080/densityMap`)
        for (let index = 0; index < response.data.length; index++) {
          var element = response.data[index];
          element.properties.color=this.getColor(element.properties.pollution)
          response.data[index]=element
          
        }
        this.mapPoll={data:response.data}
        this.time=response.data.timestamp
        
      } else {
        console.log('getting cells',this.poll)
        axios.get(`http://qaira-data.online:8080/densityMap?pollutant=${this.poll}`).then( (response) =>{
          this.mapPoll={data:response.data}
          this.time=response.data.timestamp 

        });
        
      }
    },
    async getSensors(){
      if (this.poll === 'Ninguno') {
        console.log('getting cells', this.poll)
        axios.get(`http://qaira-data.online:8080/sensors`).then((response) =>{
          this.sensors=response.data.sensors
        });
        
      } else {
        console.log('getting cells',this.poll)
        axios.get(`http://qaira-data.online:8080/sensors?pollutant=${this.poll}`).then( (response) =>{
          this.sensors=response.data.sensors


        });
        
      }

    },
    async getRoute(){
      this.loading=true
      if (this.poll==='Ninguno') {
        console.log('getting cells', this.poll)
        var response = await axios.get(`http://qaira-data.online:8080/route?startpointlat=${this.marker1[1]}&startpointlon=${this.marker1[0]}&endpointlat=${this.marker2[1]}&endpointlon=${this.marker2[0]}`)
        this.path= (response.data.paths.length!=0? {data:{type:'FeatureCollection',features:[{geometry:response.data.paths[0].points, type:"Feature"}]}}:{})
        this.instructions=(response.data.paths.length!=0? response.data.paths[0].instructions:[])
        this.accumulated=(response.data.paths.length!=0? response.data.paths[0].weight:0)
        
        this.loading=false 
        this.updated=true
      } else {
        console.log('getting cells',this.poll.value)
        var response1 = await axios.get(`http://qaira-data.online:8080/route?pollutant=${this.poll}&startpointlat=${this.marker1[1]}&startpointlon=${this.marker1[0]}&endpointlat=${this.marker2[1]}&endpointlon=${this.marker2[0]}`)
        this.path=(response1.data.paths.length!=0? {data:{type:'FeatureCollection',features:[{geometry:response1.data.paths[0].points, type:"Feature"}]}}:{})
        this.instructions=(response1.data.paths.length!=0? response1.data.paths[0].instructions:[])
        this.accumulated=(response1.data.paths.length!=0? response1.data.paths[0].weight:0)
        this.loading=false
        this.updated=true
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
      this.updated=false
    },
    marker2: function(val){
      this.marker2=val
      this.updated=false
    },

  }
}

</script>

<style scoped>
#mapContainer {
 width: 80vw;
 height: 90vh;
}
.v-icon{
  position: absolute;
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