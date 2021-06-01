<template>
  <div class="bar">
      <b-container>
          <b-row>
            <b-col>
                <h2 class="bar1">PUCP</h2>
            </b-col>
            <b-col class="topbar">
                <b-button @click="modalShow=!modalShow" variant="primary" class="button1">Info</b-button>

                <b-modal v-model="modalShow" title="Información" hide-footer>
                    <h1>Proyecto de Tesis</h1>
                    <p class="my-4">Este es el front del proyecto de Tesis</p>
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