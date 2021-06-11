<template>
  <div class="bar">
      <b-container>
          <b-row>
            <b-col>
                <h2 class="bar1">PUCP</h2>
            </b-col>
            <b-col class="topbar">
                <b-button @click="modalShow=!modalShow" variant="primary" class="button1">Información</b-button>

                <b-modal v-model="modalShow" title="Información" hide-footer>
                    <h1>Proyecto de Tesis</h1>
                    <p class="my-4">Este aplicativo utiliza información recogida por estaciones de monitoreo de aire para la planeación de rutas de aire limpio. </p>
                    <h3>¿Cómo usarlo?</h3>
                    <p class="my-4" style="text-align:justify;"> Es  muy sencillo, en primer lugar selecciona el contaminante con el que quieres calcular tu ruta. también puedes no escoger un contaminante. En este caso, se calculará la ruta más corta.<br/> Luego, arrastra los marcadores de inicio y fin hasta las ubicaciones deseadas. <br/> Finalmente,da click en buscar Ruta para que se calcule la ruta solicitada. </p>
                    <h3>¿Qué significan los colores en el mapa?</h3>
                    <p class="my-4" style="text-align:justify;" > Es  muy sencillo, estos colores representan la concentración del contaminante seleccionado. En caso no se seleccione ninguno, el color por defecto es rojo. </p>
                    <p class="my-4" style="text-align:justify;"> Estos colores están basados en el cálculo del índice de calidad del aire (<a href="https://www.minam.gob.pe/wp-content/uploads/2016/07/RM-N%c2%b0-181-2016-MINAM.pdf">INCA</a>).<br/>
                     Donde un color <a style="color:red">rojo</a>  induca una contaminación con la que se debe tener extremo cuidado. Mientras que un color <a style="color:orange">naranja</a> indica una mala calidad de aire. <br/>
                     Por otro lado, un color <a style="color:yellow">amarillo</a> indica una calidad de aire moderada. Por último, el color <a style="color:green">verde</a> indica una buena calidad del aire. </p>
                    <h3>¿Qué significan los <v-icon slot="marker" large  color="red" >mdi-crosshairs-gps</v-icon> en el mapa?</h3>      
                    <p class="my-4" style="text-align:justify;"> Es  muy sencillo, estos <v-icon slot="marker" large  color="red" >mdi-crosshairs-gps</v-icon> representan los sensores que monitorean la calidad del aire del área de estudio. Si le das click a cualquiera de ellos, mostraran informacion acerca del contaminante seleccionado. </p>
                                                      
                </b-modal>
            </b-col>
          </b-row>
      </b-container>
  </div>
</template>

<script>
export default {
    name: 'NavBar',
    data(){
        return{
            modalShow: false
        }
    }

}
</script>

<style scoped>
.bar1{
    text-align: left;
    margin: 10px;
}
.bar{
    background-color: #1B4965;
    color:#fff;
    height: 60px;
}
.topbar{
    text-align: right;
  margin-top: 10px;

}
.button1{
    margin: 0px;
}
</style>