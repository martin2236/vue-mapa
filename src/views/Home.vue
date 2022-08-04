<!-- eslint-disable prettier/prettier -->
<!-- eslint-disable prettier/prettier -->
<!-- eslint-disable prettier/prettier -->
<template>


  <div >
    <div
      class="d-flex flex-md-row flex-sm-column flex-wrap align-center justify-space-around"
      style=" background-color: #5856D6;"
        >
          
            <h1
              class="white--text "
            >El Mapa</h1>
          
              <div style="width: 500px;" class="d-flex flex-row align-center justify-center">
                <v-col cols="8">
                  <v-text-field dark v-model="busqueda"  placeholder="8001 av costanera mar del tuyú" class="custom-placeholer-color"></v-text-field>
                </v-col>
                <v-col cols="4">
                  <v-btn
                    color="#5856D6"
                    class="white--text"
                    @click="buscar">buscar
                  </v-btn>
                </v-col>
              </div>

              <div class=" d-flex align-center subir" >
                <input style="opacity:0; position: absolute; z-index:2000; width:220px;" type="file" name="xlfile" id="xlf" v-on:change="onChange($event)" />
                <v-btn color='#5856D6' class="white--text mt-2"><v-icon class="mr-1">mdi-paperclip</v-icon> subir archivo</v-btn>
              </div>

      </div>
        <div style="height: 130px; overflow: auto;" class="text-center">
        <div style="margin-top:20px;">

       <button class="boton mb-2 mt-lg-2" style="width:165px" @click="marcarUbicacion = !marcarUbicacion">
        marcar ubicaciones 
      </button>
      <button class="boton mb-2 mt-lg-2" style="width:165px" @click="borrarUbicacion()">
        borrar ubicaciones
      </button>
       <button class="boton mb-2 mt-lg-2" style="width:165px" @click="marcarPoligono = !marcarPoligono">
        marcar poligono
      </button>
      <button class="boton mb-2 mt-lg-2" style="width:165px" @click="polygon.latlngs=[]">
        borrar poligono
      </button>
     </div>
     <div id="app">
      <v-btn v-if="alumnos != ''" @click="dialog = !dialog" color="#5856D6" class=" white--text">ver alumnos</v-btn>
    </div>
     
    </div>
        <div style="position: absolute; top:100; width:100%">
          <l-map
          class="mx-auto"
          id="map"
          v-if="showMap"
          :zoom="zoom"
          :center="center"
          :options="mapOptions"
          style="height: 500px ; width:90vw; border:2px solid #5856D6;"
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

      <v-row justify="center">
    <v-dialog
      v-model="dialog"
      fullscreen
      hide-overlay
      transition="dialog-bottom-transition"
    >
      
      <v-card>
        <v-toolbar
          dark
          color="#5856D6"
        >
          <v-btn
            icon
            dark
            @click="dialog = false"
          >
            <v-icon>mdi-close</v-icon>
          </v-btn>
          <v-toolbar-title>Volver</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-toolbar-items>
            <v-btn
              dark
              text
              @click="dialog = false"
            >
              Añadir registro
            </v-btn>
          </v-toolbar-items>
        </v-toolbar>
        <v-card>
        <v-card-title>
          Alumnos
          <v-spacer></v-spacer>
          <v-text-field
            v-model="search"
            append-icon="mdi-magnify"
            label="Search"
            single-line
            hide-details
          ></v-text-field>
          </v-card-title>
            <v-data-table
              :headers="headers"
              :items="alumnos"
              :search="search"
          >
          <template v-slot:[`item.coord`]="{item}">
              <v-icon :color="item != '' ? 'error' : 'mdi-check-circle-outline'">
                {{item != '' ? "mdi-close-circle-outline" : "mdi-check-circle-outline"}}
              </v-icon>
          </template>
          </v-data-table>
      </v-card>
      </v-card>
    </v-dialog>
  </v-row>

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
      dialog: false,
      search: "",
      headers: [
        {
          text: "NOMBRE",
          align: "start",
          sortable: false,
          value: "alumno",
        },
        { text: "TELEFONO", value: "telefono" },
        { text: "FECHA_NAC", value: "cumpleanos" },
        { text: "DIRECCION", value: "lugar" },
        { text: "COORDENADAS", value: "coord", align: "center" },
      ],
      alumnos: "",
      file: null,
      metaDataFile: null,
      data: null,
      busqueda: "",
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
      const file = e.target.files[0];
      /* Boilerplate to set up FileReader */
      const reader = new FileReader();
      reader.onload = (e) => {
        /* Parse data */
        const bstr = e.target.result;
        const wb = XLSX.read(bstr, { type: "binary" });
        /* Get first worksheet */
        const wsname = wb.SheetNames[0];
        const ws = wb.Sheets[wsname];
        /* Convert array of arrays */
        let rowObject = XLSX.utils.sheet_to_row_object_array(ws);
        /* Update state */
        this.data = rowObject;
        this.getDirecciones();
      };
      reader.readAsBinaryString(file);
    },
    getDirecciones() {
      let direcciones = this.data.map((item) => {
        let modificar = item.__EMPTY_4.split(" ");
        let direccion =
          modificar[1] +
          " " +
          modificar[0] +
          " " +
          item.__EMPTY_5.toLowerCase();
        return {
          alumno: item.__EMPTY.toLowerCase() + " " + item.__EMPTY_1.toLowerCase(),
          telefono: item.__EMPTY_2,
          cumpleanos: item.__EMPTY_8,
          lugar: direccion + " Argentina",
          coord: '',
        };
      });
      let listaFiltrada = direcciones.slice(1);
      this.alumnos = listaFiltrada;
      // console.log(listaFiltrada);
      // listaFiltrada.forEach((item) => {
      //   setTimeout(() => {
      //     fetch(
      //       `https://us1.locationiq.com/v1/search?key=pk.fde25416315bf622505a201e1dbfbacb&q=${item.lugar}&format=json`
      //     )
      //       .then((res) => {
      //         return res.json();
      //       })
      //       .then((data) => {
      //         console.log(data);
      //         // if (data[0].lat != undefined) {
      //         //   this.ubicacionesSeleccionadas.push({
      //         //     lugar: latLng(data[0].lat, data[0].lon),
      //         //     nombre: item.alumno + data[0].lat + data[0].lon,
      //         //   });
      //         //     if(this.ubicacionesSeleccionadas.length == listaFiltrada.length){
      //         //     console.log(this.ubicacionesSeleccionadas)
      //         //     this.showMap = false
      //         //   }
      //         // }
      //       });
      //   }, 3000);
      // });
    },
    buscar() {
      let q = this.busqueda + " argentina";
      console.log(q);
      fetch(
        `https://us1.locationiq.com/v1/search?key=pk.fde25416315bf622505a201e1dbfbacb&q=${q}&format=json`
      )
        .then((res) => {
          return res.json();
        })
        .then((data) => {
          console.log(data);
          this.center = latLng(data[0].lat, data[0].lon);
          this.ubicacionesSeleccionadas.push({
            lugar: latLng(data[0].lat, data[0].lon),
          });
          this.showMap = false;
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
@media(max-width: 412px){
  .subir{
      display: none !important;
  }
}

.custom-placeholer-color input::placeholder {
  color: red!important;
  opacity: 1;
}

.custom-label-color .v-label {
  color: red;
  opacity: 1;
}

.custom-placeholer-color input,
.custom-label-color input{
  color: red!important;
}

.boton {
  margin-right: 20px;
  background-color: #5856d6;
  color: #fff;
  padding: 10px;
}
#map { 
  z-index: 1; 
}
</style>
