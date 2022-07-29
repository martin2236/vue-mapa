<!-- eslint-disable prettier/prettier -->
<!-- eslint-disable prettier/prettier -->
<!-- eslint-disable prettier/prettier -->
<template>

  <div style="height: 80vh; width: 100%; display:flex; flex-direction:column; align-items:center;">
    <div style="height: 400px; overflow: auto;" class="text-center">
     <div class="d-flex flex-row align-center">
      <v-col cols="8">
        <v-text-field v-model="busqueda" class="buscador"></v-text-field>
      </v-col>
      <v-col cols="4">
        <v-btn
          color="#5856D6"
          class="white--text"
         @click="buscar">buscar</v-btn>
      </v-col>
     </div>
     <div id="app">
    <v-file-input
    v-on:change="onChange($event)"
    v-model="file">
    </v-file-input>
    <input type="file" name="xlfile" id="xlf" v-on:change="onChange($event)" />
  </div>
      <!-- <p>centrado en {{ center }} el zoom es: {{ currentZoom }}</p>
      <p>El margen de error es de {{ margen }} metros</p> -->
      <button class="boton" @click="marcarUbicacion = !marcarUbicacion">
        marcar ubicaciones 
      </button>
      <button class="boton" @click="borrarUbicacion()">
        borrar ubicaciones
      </button>
       <button class="boton" @click="marcarPoligono = !marcarPoligono">
        marcar poligono
      </button>
      <button class="boton" @click="polygon.latlngs=[]">
        borrar poligono
      </button>
    </div>
    <l-map
      v-if="showMap"
      :zoom="zoom"
      :center="center"
      :options="mapOptions"
      style="margin-top:100px; height: 90% ; width:90%; border:2px solid #5856D6"
      @update:center="centerUpdate"
      @update:zoom="zoomUpdate"
      @click="marcarUbicacion ? click($event) : nada(),marcarPoligono ? poligono($event) : nada()"
    >
    <l-polygon :lat-lngs="polygon.latlngs" :color="polygon.color"></l-polygon>
      <l-tile-layer
        :url="url"
        :attribution="attribution"
      />
      <l-marker v-for="(ubicacion, index) in ubicacionesSeleccionadas" :key="index" :lat-lng="ubicacion.lugar">
        <l-popup>
          <div @click="innerClick">
            {{ubicacion.nombre}}
            <p v-show="showParagraph">
              Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque
              sed pretium nisl, ut sagittis sapien. Sed vel sollicitudin nisi.
              Donec finibus semper metus id malesuada.
            </p>
          </div>
        </l-popup>
      </l-marker>
      
    </l-map>
  </div>
</template>

<script>
import { latLng } from "leaflet";
import { LMap, LTileLayer, LMarker, LPopup, LPolygon } from "vue2-leaflet";
var XLSX = require("xlsx");

export default {
  name: "Example",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    LPolygon,
  },
  data() {
    return {
      file:null,
      metaDataFile: null,
      data:null,
      busqueda:'',
      zoom: 13,
      center: latLng(-36.735349, -56.6895547),
      margen: "",
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      withPopup: latLng(-36.735349, -56.6895547),
      withTooltip: latLng(47.41422, -1.250482),
      currentZoom: 11.5,
      currentCenter: latLng(47.41322, -1.219482),
      showParagraph: false,
      ubicacionesSeleccionadas: [],
      marcarUbicacion: false,
      marcarPoligono: false,
      coordOptions: {
        enableHighAccuracy: true,
        timeout: 5000,
        maximumAge: 0,
      },
      mapOptions: {
        zoomSnap: 0.5,
      },
      polygon: {
        latlngs: [],
        color: "green",
      },
      showMap: true,
    };
  },
  created() {
    this.getCorrds();
  },
  methods: {
     onChange(e) {
      const file = e.target.files[0]
      /* Boilerplate to set up FileReader */
      const reader = new FileReader()
      reader.onload = (e) => {
        /* Parse data */
        const bstr = e.target.result
        const wb = XLSX.read(bstr, { type: 'binary' })
        /* Get first worksheet */
        const wsname = wb.SheetNames[0]
        const ws = wb.Sheets[wsname]
        /* Convert array of arrays */
        let rowObject = XLSX.utils.sheet_to_row_object_array(ws)
        /* Update state */
        this.data = rowObject
        this.getDirecciones()
      }
      reader.readAsBinaryString(file)
    },
    getDirecciones(){
     let direcciones = this.data.map(item=>{return {alumno:item.__EMPTY +' '+ item.__EMPTY_1,lugar:item.__EMPTY_4 +' '+ item.__EMPTY_5+' '+'La Costa'+' '+'Argentina'}})
     let listaFiltrada = direcciones.slice(1)
     listaFiltrada.forEach(item => {
       fetch(
        `http://api.positionstack.com/v1/forward?access_key=30dc4943e77b8d3c2d24f20b135e403e&query=${item.lugar}`
      )
        .then((res) => {
          return res.json();
        })
        .then((data) => {
          if(data.data[0].latitude != undefined){
            console.log(item.alumno)
            console.log('latitud',data.data[0].latitude, 'longitud', data.data[0].longitude);
             this.ubicacionesSeleccionadas.push({lugar: latLng(data.data[0].latitude, data.data[0].longitude),nombre:item.alumno});
          }
         
        });
     })
    },
    buscar() {
      console.log(this.busqueda)
      fetch(
        `http://api.positionstack.com/v1/forward?access_key=30dc4943e77b8d3c2d24f20b135e403e&query=${this.busqueda}`
      )
        .then((res) => {
          return res.json();
        })
        .then((data) => {
          console.log('latitud',data.data[0].latitude, 'longitud', data.data[0].longitude);
          this.center = latLng(data.data[0].latitude, data.data[0].longitude);
          this.ubicacionesSeleccionadas.push({lugar: latLng(data.data[0].latitude, data.data[0].longitude)});
        });
    },
    borrarUbicacion() {
      this.ubicacionesSeleccionadas = [];
    },
    click(e) {
      var coord = e.latlng;
      this.ubicacionesSeleccionadas.push({
        lugar: latLng(coord.lat, coord.lng),
        titulo: "",
        descripcion: "",
      });
      console.log(this.ubicacionesSeleccionadas);
    },
    poligono(e) {
      var coord = e.latlng;
      this.polygon.latlngs.push([coord.lat, coord.lng]);
      console.log(this.ubicacionesSeleccionadas);
    },
    nada() {
      console.log("nada");
    },
    getCorrds() {
      navigator.geolocation.getCurrentPosition(
        this.success,
        this.error,
        this.coordOptions
      );
    },
    success(pos) {
      const crd = pos.coords;
      this.margen = crd.accuracy;
      this.center = latLng(crd.latitude, crd.longitude);
    },
    error(err) {
      console.warn(`ERROR(${err.code}): ${err.message}`);
    },
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.currentCenter = center;
    },
    showLongText() {
      this.showParagraph = !this.showParagraph;
    },
    innerClick() {
      alert("Click!");
    },
  },
};
</script>
<style scoped>
.boton {
  margin-right: 20px;
  background-color: #5856d6;
  color: #fff;
  padding: 10px;
}
</style>
