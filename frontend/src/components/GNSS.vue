<template>
  <div id="tab-window">
    <div>

      <h3 style="text-align: center">GNSS Data Analysis</h3>
      <label class="control-label requiredField">
        Select GNSS data models
      </label>
  

        <!-- <label for="sel1">Select list:</label> -->
        <select class="form-control" v-model="kmltype_sel" id="kmltype_sel" >
          <option value='getvelocities'>Velocities</option>
          <option value='getcoseismic'>Coseismic</option>
          <option value='getpostseismic'>Postseismic</option>
          <option value='getdisplacement'>Displacement</option>
          <option value='getmodel'>Model</option>
        </select>

        <b-button variant="dark" id="sp_windowpicker" class="btn btn-light" @click="gnssDrawRect()">
          <b-icon-pencil></b-icon-pencil> Draw an area on map</b-button>
        <b-button variant="warning" id="clearGnss" @click="clearGnss()"><b-icon-trash></b-icon-trash> Clear GNSS Layers</b-button>
        <br/>

        <div v-if="geometryActive" >
          <br/>
          <b-button variant="warning" @click="drawListenerOff">
            <b-icon-x-circle></b-icon-x-circle>Cancel Selection</b-button>
          <br/>
        </div>
        <br>
        <b-input-group prepend="Center Latitude">
          <template #prepend><b-input-group-text ><strong>Center Latitude</strong></b-input-group-text></template>
          <b-form-input v-model="gs_latitude"  name="gs_latitude"></b-form-input>
        </b-input-group>

        <b-input-group prepend="Center Longitude">
          <template #prepend><b-input-group-text ><strong>Center Longitude</strong></b-input-group-text></template>
          <b-form-input v-model="gs_longitude" placeholder="" name="gs_longitude"></b-form-input>
        </b-input-group>

        <b-input-group prepend="Longitude Span">
          <template #prepend><b-input-group-text ><strong>Longitude Span</strong></b-input-group-text></template>
          <b-form-input v-model="gs_width"  name="gs_width" placeholder="1 degree"></b-form-input>
        </b-input-group>

        <b-input-group prepend="Latitude Span">
          <template #prepend><b-input-group-text ><strong>Latitude Span</strong></b-input-group-text></template>
          <b-form-input v-model="gs_height" placeholder="1 degree" name="gs_height"></b-form-input>
        </b-input-group>

        <div class="input-group" id="epoch_show" v-if="this.kmltype_sel === 'getcoseismic' || this.kmltype_sel === 'getpostseismic'">
          <b-input-group prepend="Epoch">
            <template #prepend><b-input-group-text ><strong>Epoch</strong></b-input-group-text></template>
            <b-form-input v-model="gs_epoch" placeholder="YYYY-MM-DD" name="gs_epoch"></b-form-input>
          </b-input-group>
        </div>

        <b-input-group prepend="Epoch 1" v-if="this.kmltype_sel === 'getdisplacement' || this.kmltype_sel === 'getmodel'">
            <template #prepend><b-input-group-text ><strong>Epoch 1</strong></b-input-group-text></template>
          <b-form-input v-model="gs_epoch1" placeholder="YYYY-MM-DD" name="gs_epoch1"></b-form-input>
        </b-input-group>

        <b-input-group prepend="Epoch 2" v-if="this.kmltype_sel === 'getdisplacement' || this.kmltype_sel === 'getmodel'">
            <template #prepend><b-input-group-text ><strong>Epoch 2</strong></b-input-group-text></template>
          <b-form-input v-model="gs_epoch2" placeholder="YYYY-MM-DD" name="gs_epoch2"></b-form-input>
        </b-input-group>

        <b-input-group prepend="Ref. Site">
          <b-form-input v-model="gs_refsite" placeholder="4-letter code" name="gs_refsite"></b-form-input>
          <b-input-group-append>
            <b-button variant="outline-primary" href="https://sideshow.jpl.nasa.gov/post/tables/table2.html" target="_blank">Stations</b-button>
          </b-input-group-append>
        </b-input-group>

        <b-input-group prepend="Scale">
          <b-form-input v-model="gs_scale" placeholder="320 mm/yr/deg" name="gs_scale"></b-form-input>
        </b-input-group>

        <b-input-group prepend="Coseismic Win." v-if="this.kmltype_sel === 'getcoseismic' || this.kmltype_sel === 'getpostseismic'">
          <b-form-input v-model="gs_ctwin" name="gs_ctwin" placeholder="0.1 years"></b-form-input>
        </b-input-group>

        <b-input-group prepend="Postseismic Win." v-if="this.kmltype_sel === 'getpostseismic' ">
          <b-form-input v-model="gs_ptwin" name="gs_ptwin" placeholder="2 years"></b-form-input>
        </b-input-group>

        <b-input-group prepend="Av. Win. 1" v-if="this.kmltype_sel === 'getdisplacement'">
          <b-form-input v-model="gs_dwin1" name="gs_dwin1" placeholder="10 days"></b-form-input>
        </b-input-group>

        <b-input-group prepend="Av. Win. 2" v-if="this.kmltype_sel === 'getdisplacement'">
          <b-form-input v-model="gs_dwin2" name="gs_dwin2" placeholder="10 days"></b-form-input>
        </b-input-group>

        <b-input-group prepend="Output Prefix">
          <b-form-input v-model="gs_outputprefix" name="gs_outputprefix"></b-form-input>
        </b-input-group>
        <b-col class="miscOptions">
          <b-row class="checkbox" style="text-align: left" v-if="this.kmltype_sel === 'getdisplacement'">
            <label class="checkbox" >
              <input v-model="gs_analysisCenter" name="analysisCenter" type="checkbox" id="gs_analysisCenter"/>
              Use NGL data
            </label>
          </b-row>
          <b-row class="checkbox" style="text-align: left">
            <label class="checkbox" >
              <input v-model="markerSize" name="vabs" type="checkbox" id="markerSize"/>
              Minimize Marker Size
            </label>
          </b-row>
          <b-row class="checkbox" style="text-align: left">
            <label class="checkbox">
              <input v-model="gs_vabs" name="vabs" type="checkbox" id="gs_vabs" value= ""/>
              Display absolute verticals
            </label>
          </b-row>
          <b-row class="checkbox" style="text-align: left">
            <label class="checkbox">
              <input v-model="gs_eon" name="mon" type="checkbox" id="gs_eon" value=""/>
              Include error ellipses
            </label>
          </b-row>
          <b-row class="checkbox" style="text-align: left" v-if="this.kmltype_sel === 'getdisplacement'">
            <label class="checkbox" >
              <input v-model="gs_interpolation" name="gsinterpolation" type="checkbox" id="gs_interpolation"/>
              Interpolation
            </label>
          </b-row>
        <b-container v-if="gs_interpolation && this.kmltype_sel === 'getdisplacement'" fluid="lg">
        <b-row>
        <b-input-group prepend="Mehtods">
          <b-form-select  v-model="gs_interpolationtype" name="gs_interpolationtype">
            <b-form-select-option value="linear">linear</b-form-select-option>
            <b-form-select-option value="gaussian">gaussian</b-form-select-option>
            <b-form-select-option value="power">power</b-form-select-option>
            <b-form-select-option value="exponential">exponential</b-form-select-option>
            <b-form-select-option value="spherical">spherical</b-form-select-option>
          </b-form-select>
        </b-input-group>
        </b-row>
        <b-row>
        <b-input-group prepend="Grid spacing">
          <b-form-input v-model="gs_gridspacing" name="gs_gridspacing" placeholder="0.018 degree"></b-form-input>
        </b-input-group>
        </b-row>
        <b-row>
        <b-input-group prepend="Azimuth">
          <b-form-input v-model="gs_azimuth" name="gs_azimuth" placeholder="-5 degree"></b-form-input>
        </b-input-group>
        </b-row>
        <b-row>
        <b-input-group prepend="Elevation Angle">
          <b-form-input v-model="gs_elevationangle" name="gs_elevationangle" placeholder="60 degree"></b-form-input>
        </b-input-group>
        </b-row>
        </b-container>

          <b-row>
            <button  class="btn btn-success" id="gs_submit" name="submit" type="submit" v-on:click.prevent="runButtonClick()">        Run
            </button>
          </b-row>
          <br />
          <b-row>
            <div style="float: left; text-align: left">Data source: <br/><a href="https://sideshow.jpl.nasa.gov/post/series.html" target="_blank">GNSS Time Series</a>
            <br><a v-if="this.kmltype_sel === 'getdisplacement'" href="http://geodesy.unr.edu/NGLStationPages/gpsnetmap/GPSNetMap.html" target="_blank">NGL GPS Networks</a>
          </div>
          </b-row>
        </b-col>

        <b-col >
          <br><br>
          <div class="outputLayers" v-if="gnssLayers.length!==0 && !activeGnssQuery">
            <strong>Output</strong>
            <div  v-for="layer in gnssLayers" :key="layer.name">
              <div v-if="layer.type !== 'table.txt' && layer.type !=='output.zip'" ><input type="checkbox" :value="layer.active" v-model="layer.active" @change="showHideLayers(layer.active, layer)"> <span class="checkbox-label"> <a target="_blank" :href="layer.url">{{layer.pre}} {{layer.type}}</a> </span> </div>
              <div v-else><a target="_blank" :href="layer.url">{{layer.name}}</a></div>
            </div>
          </div>

        </b-col>

      <div v-if="activeGnssQuery" style="overflow: hidden">
        <br/>
        <b-spinner variant="success" label="Spinning"></b-spinner>
      </div>
      <br/>



    </div>

  </div>


</template>

<script>

import {bus} from '@/main'
import axios from 'axios'
import { mapFields } from 'vuex-map-fields';
import L from "leaflet";

export default {

  name: "GNSS-tools",
  data() {
    return {
      areaLayer: null,
      gs_interpolationtype:'linear',
    }
  },
  computed: {
    ...mapFields([
      'gnss.selected',
      'gnss.kmltype_sel',
      'gnss.gs_latitude',
      'gnss.gs_longitude',
      'gnss.gs_width',
      'gnss.gs_height',
      'gnss.gs_epoch',
      'gnss.gs_epoch1',
      'gnss.gs_epoch2',
      'gnss.gs_refsite',
      'gnss.gs_scale',
      'gnss.gs_ctwin',
      'gnss.gs_ptwin',
      'gnss.gs_dwin1',
      'gnss.gs_dwin2',
      'gnss.gs_outputprefix',
      'gnss.kmlData',
      'gnss.gs_eon',
      'gnss.gs_vabs',
      'gnss.gs_analysisCenter',
      'gnss.gs_interpolation',
      'gnss.gs_gridspacing',
      'gnss.gs_interpolationtype',
      'gnss.gs_azimuth',
      'gnss.gs_elevationangle',

      'gnss.ranLayers',
      'gnss.activeLayers',
      'gnss.markerSize',
      'gnss.layersActive',
      'gnss.activeGnssQuery',
      'gnss.geometryActive',
      'gnss.gnssLayers',
      'gnss.interpolationLegend',
        'map.drawControl',
        'map.globalMap',
        'map.layers',
    ])

  },
  mounted() {
    bus.$on('gnssDrawQuery', (maxLat, minLon, minLat, maxLon, centerLat, centerLng) =>
        this.setRect(maxLat, minLon, minLat, maxLon, centerLat, centerLng));
  },

  methods: {
    showHideLayers(active, layer){
      let name = layer.pre + layer.type;
        if(active){
          this.globalMap.addLayer(this.layers[name])
        }else {
          this.globalMap.removeLayer(this.layers[name]);
        }
        if ((active) && (layer.type.includes(".png"))) {
          // add legend
          this.interpolationLegend.update(layer.url);
        } else {
          // remove legend
          this.interpolationLegend.remove();
        }
      },
    runButtonClick(){
      let vm = this;
      if(vm.areaLayer!=null){
        vm.globalMap.removeLayer(vm.areaLayer);
        vm.areaLayer=null;
      }
      this.rungpsservice();
    },
    rungpsservice(){
      this.activeGnssQuery = true;
      var vm = this;
      var fileNameH, fileNameV, fileNameT, folder, props,fileNameZ;
      //var markerSize = this.markerSize;
      var verticalUrl, horizontalUrl, tableUrl, zipUrl;
      var prefix = this.gs_outputprefix;
      if(this.kmltype_sel === ''){
        alert("Please select as least one plot!");
      }
      else {

        for(var i = 0;i<this.gnssLayers.length;i++){
          var splitPrefix = this.gnssLayers[i].name.split('_')[0];
          if(splitPrefix === this.gs_outputprefix){
            continue;
            //alert('There is already an existing query with that name, please rename and resubmit');
            //return;
          }
        }
        // this.layerCheckbox = true;
        if (this.gs_analysisCenter == true) {this.gs_analysisCenter = "NGL";} else { this.gs_analysisCenter = "";}
        const baseURI = '/geogateway_django_app/gps_service'
        //request JSON dict of GPS_service details with query params from form
        var gpsfunction = this.kmltype_sel;
        if ((gpsfunction == "getdisplacement") && (this.gs_interpolation == true)) {gpsfunction = "getInterpolation";}
        axios.get(baseURI, {
          params: {
            //
            "function": gpsfunction,
            "lat": this.gs_latitude,
            "lon": this.gs_longitude,
            //"width":$('#gs_width').val(),
            //"height":$('#gs_height').val(),
            "width": this.gs_width,
            "height": this.gs_height,
            "epoch": this.gs_epoch,
            "epoch1": this.gs_epoch1,
            "epoch2": this.gs_epoch2,
            "scale": this.gs_scale,
            "ref": this.gs_refsite,
            "ct": this.gs_ctwin,
            "pt": this.gs_ptwin,
            "dwin1": this.gs_dwin1,
            "dwin2": this.gs_dwin2,
            "prefix": this.gs_outputprefix,
            //need default false value?
            "mon":this.markerSize,
            "eon": this.gs_eon,
            "vabs": this.gs_vabs,
            "analysisCenter": this.gs_analysisCenter,
            "gridspacing": this.gs_gridspacing,
            "interpolationtype": this.gs_interpolationtype,
            "azimuth": this.gs_azimuth,
            "elevation": this.gs_elevationangle,
            //
          }
        })
            //use JSON results (filename and folder) to request raw kml text
            .then(function (response) {
              props = response.data;
              console.log(props);
              if (!(typeof props === 'object')) {
                vm.activeGnssQuery = false;
                alert("Somthing wrong, please check input paramters!");
                return;
              }
              function getExtension(f) {
                var parts = f.split('_');
                return parts[parts.length - 1];
              }
              var imagelist = [];
              var hasZip = false;
              for(var i = 0;i < props.urls.length;i++){
                var ext = getExtension(props.urls[i]);
                if(ext == 'vertical.kml'){
                  verticalUrl = props.urls[i];
                  fileNameV = props.results[i];
                }else if(ext == 'horizontal.kml'){
                  horizontalUrl = props.urls[i];
                  fileNameH = props.results[i];
                }else if(ext == 'table.txt'){
                  tableUrl = props.urls[i];
                  fileNameT = props.results[i];
                }else if(!ext.includes('colorbar') && ext.includes('.png')) {
                  imagelist.push([props.urls[i],props.results[i]]);
                }else if (ext.includes(".zip")) {
                  hasZip = true;
                  zipUrl = props.urls[i];
                  fileNameZ = props.results[i];
                }
              }
              //console.log(imagelist);
              //console.log(props.imagebounds);


              folder = props.folder;
              prefix = (1 + Math.floor(vm.gnssLayers.length/3)).toString() + prefix;
              vm.gnssLayers.push({
                pre: prefix,
                name: fileNameT,
                folder: folder,
                active: true,
                url: tableUrl,
                type: 'table.txt',
              })
              vm.gnssLayers.push({
                pre: prefix,
                name: fileNameH,
                folder: folder,
                active: true,
                url: horizontalUrl,
                type: 'horizontal.kml',
              })
              vm.gnssLayers.push({
                pre: prefix,
                name: fileNameV,
                folder: folder,
                active: true,
                url: verticalUrl,
                type: 'vertical.kml',
              })
            if (hasZip) {
              vm.gnssLayers.push({
                pre: prefix,
                name: fileNameZ,
                folder: folder,
                active: true,
                url: zipUrl,
                type: 'output.zip',
              });              
            }
            if (imagelist.length>0){
              for (var j = 0;j < imagelist.length;j++) {
                var iname = imagelist[j][1].replace("contour_of_","");
                var activestatus = false;
                if (j==0) {activestatus=true;}
              vm.gnssLayers.push({
                pre: prefix,
                name: iname,
                folder: folder,
                active: activestatus,
                url: imagelist[j][0],
                type: iname,
              });
              vm.addImageLayer(imagelist[j][0],props.imagebounds,prefix+iname,activestatus);
              }
            }
            imagelist=[];
              const kmlURI = '/geogateway_django_app/get_kml';
              axios.get(kmlURI, {
                params: {
                  "file": fileNameH,
                  "folder": folder
                },
                responseType: 'text',
                //emit raw kml text to parent map component
              }).then(function (response) {
                // console.log(toGeoJSON.kml(response.data));
                let hName = prefix + 'horizontal.kml';
                vm.addGnssLayer(response.data, hName);

              })
              axios.get(kmlURI, {
                params: {
                  "file": fileNameV,
                  "folder": folder
                },
                responseType: 'text',
                //emit raw kml text to parent map component
              }).then(function (response) {
                // console.log(toGeoJSON.kml(response.data));
                // var geojson = toGeoJSON.kml((new DOMParser()).parseFromString(response.data, 'text/xml'))
                // console.log(geojson)
                let vName = prefix + 'vertical.kml';
                vm.addGnssLayer(response.data, vName);
                //console.log(markerSize)
                vm.activeGnssQuery = false;
              })
            })
        this.layersActive = true;
      }

    },
    addGnssLayer(file, type) {
      this.kmlText(file, type);
    },
    kmlText(text, layerName) {
      const parser = new DOMParser();
      const kml = parser.parseFromString(text, 'text/xml');
      this.layers[layerName] = new L.KML(kml);
      this.globalMap.addLayer(this.layers[layerName]);

    },
    addImageLayer(imageUrl,imageBounds,layerName,displayflag){
      this.layers[layerName]=new L.imageOverlay(imageUrl, imageBounds,{opacity:0.85});
      if (displayflag) {
      this.globalMap.addLayer(this.layers[layerName]);
      this.globalMap.flyToBounds(this.layers[layerName].getBounds());
      this.addImageLegend(imageUrl); }
    },
    addImageLegend(aimageUrl){
      var legendUrl = aimageUrl.replace(".png","_colorbar.png");
      if (this.interpolationLegend == null) {
        this.interpolationLegend = L.control({position: 'bottomright'});
        this.interpolationLegend.onAdd = function () {
          var div = L.DomUtil.create('div', 'leafletinfo');
          div.id = "interpolationLegend";
          div.innerHTML = '<img src=' + legendUrl + '>' ;
          return div;
        };
        this.interpolationLegend.update = function (aimageUrl) {
          var aurl = aimageUrl.replace(".png","_colorbar.png");
          var div = document.getElementById('interpolationLegend');
          div.innerHTML = '<img src=' + aurl + '>' ;
        }
        this.interpolationLegend.remove = function () {
          var div = document.getElementById('interpolationLegend');
          if (div != null) {
          div.innerHTML = '';}
        }
        this.interpolationLegend.addTo(this.globalMap);
      } else {this.interpolationLegend.update(aimageUrl);}
        
    },
    drawToolbar() {
      this.geometryActive = true;

      new L.Draw.Rectangle(this.globalMap, this.drawControl.options.rectangle).enable();

      this.drawListener('gnss');
    },
    gnssDrawRect(){
      this.geometryActive = true;
      let vm = this;
      vm.rectDraw = new L.Draw.Rectangle(vm.globalMap, vm.drawControl.options.rectangle);
      vm.rectDraw.enable();
      vm.globalMap.on('draw:created', function (e) {
        if(vm.areaLayer!=null){
          vm.globalMap.removeLayer(vm.areaLayer)
        }
        var type = e.layerType;
        if (type === 'rectangle') {
          var layer = e.layer;
          vm.globalMap.addLayer(layer);
          vm.centerLat = layer.getCenter().lat;
          vm.centerLng = layer.getCenter().lng;
          vm.maxLat = layer.getLatLngs()[0][1].lat;
          vm.maxLon = layer.getLatLngs()[0][2].lng;
          vm.minLat = layer.getLatLngs()[0][3].lat;
          vm.minLon = layer.getLatLngs()[0][0].lng;
          vm.areaLayer=layer;
          vm.rectDraw = null;
          bus.$emit('gnssDrawQuery', vm.maxLat, vm.minLon, vm.minLat, vm.maxLon, vm.centerLat, vm.centerLng)
          vm.geometryActive = false;
        }});

    },
    drawListenerOff(){
      this.geometryActive = false;

      this.rectDraw.disable();
    },
    clearGnss(){
      let vm = this;
      if(vm.areaLayer!=null){
        vm.globalMap.removeLayer(vm.areaLayer);
        vm.areaLayer=null;
      }
      this.layersActive = false;
      for(var i = 0; i < this.gnssLayers.length; i++){
        let curr = this.gnssLayers[i];
        if ((curr.type !== 'table.txt') && (curr.type !=='output.zip')) {
          let name = curr.pre + curr.type;
          //console.log("remove layer " + name);
          this.globalMap.removeLayer(this.layers[name]);
        }
      }
      // remove gnss legend layer
      if (this.interpolationLegend !== null) {
          this.interpolationLegend.remove();
        }
      this.interpolationLegend == null;

      this.gs_latitude =null;
      this.gs_longitude = null;
      this.gs_width = null;
      this.gs_height = null;
      this.gnssLayers = [];
    },
    setRect(maxLat, minLon, minLat, maxLon, centerLat, centerLng){
      this.gs_latitude = centerLat;
      this.gs_longitude = centerLng;
      this.gs_height = Math.abs(maxLat - minLat);
      this.gs_width = Math.abs(maxLon - minLon);
    }

  },
}
</script>

<style scoped>

.input-group {
  width: 100%;
}
strong {
  color: #343a40;
}

.miscOptions {
  float: left;
  width: 50%;
}

.outputLayers {
  /*color: #343a40;*/
  margin-left: 55%;
  font-size: 14px;
  width: 40%;
  border: 2px solid #416c41;
  box-sizing: border-box;
  border-radius: 0px;
  background-color: #bad7ff;
  text-align: left;
  margin-right: auto;
  padding:5px;
  position: absolute;
}

a:link {
  color: black;
  background-color: transparent;
  text-decoration: underline;
}

</style>
<style>
.leafletinfo{
    padding: 6px 8px;
    font: 14px/16px Arial, Helvetica, sans-serif;
    background: white;
    background: rgba(255,255,255,0.6);
    box-shadow: 0 0 15px rgba(0,0,0,0.2);
    border-radius: 5px;
}

</style>
