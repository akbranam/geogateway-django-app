<template>
  <div id="map-window">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css"
          integrity="sha512-xwE/Az9zrjBIphAcBb3F6JVqxf46+CDLwfLMHloNu6KEQCAWi6HcDUbeOfBIptF7tcCzusKFjFw2yuvEpDL9wQ=="
          crossorigin=""/>

    <TopNav/>
    <ToolBar/>
    <DraggableDiv v-resize @resize="resizeLOS" class="col-11" v-if="plotActive" id="plot-window">
      <vue-resize ></vue-resize>
      <template slot="header">
        <p style="color: #000000">Line of Sight Displacement</p>
      </template>
      <div id="losLegend">
      </div>
        <template slot="main" >
        <div id="dygraph-LOS" v-bind:style="losStyle"></div>
      </template>

    </DraggableDiv>
    <!--        <div v-if="plotActive" class="plot-window">-->
    <!--            <div id="los-header"><h4>LOS Plot</h4></div>-->
    <!--            <div id="dygraph-LOS"></div>-->
    <!--        </div>-->

    <div id="map">
   </div>



  </div>
</template>

<script>

import 'leaflet/dist/leaflet.css';
import L from 'leaflet';
import 'leaflet-kml'
import ToolBar from "./ToolBar";
import TopNav from "./TopNav";
import {bus} from '../main'
import 'leaflet-draw'
import "leaflet-draw/dist/leaflet.draw.css";

import {circleMaker, gdacsPopup, popupMaker} from '../assets/mapMethods'
import Dygraph from "dygraphs";
import 'dygraphs/dist/dygraph.css';
import DraggableDiv from "./DraggableDiv";
import 'leaflet-kmz';
import { mapFields } from 'vuex-map-fields';
import 'vue-resize/dist/vue-resize.css'
import VueResize from 'vue-resize';
// import axios from "axios";
// import GeometryUtil from 'leaflet-geometryutil'


export default {
  name: 'MyMap',
  components: {
    ToolBar,
    TopNav,
    DraggableDiv,
    VueResize,
  },
  data() {
    return {
      drawToolShow: false,

      kmlLayers: [],
      gnssV: [],
      gnssH: [],

      usgsLegend: null,
      centerLat: null,
      centerLng: null,
      maxLat: null,
      maxLon: null,
      minLat: null,
      minLon: null,
      plotResize: null,
      losPlot: null,
      losStyle: {
        height: '250px',
        width: '675px',
        marginLeft: '5px',
        marginBottom: '5px',
        borderColor: '#5cb85c'
      }


    };
  },
  computed: {
    ...mapFields(['uavsar.uavsarLayers', 'map.globalMap', 'map.layers', 'map.drawControl'
      ,'map.uavsarLegend', 'map.plotActive', 'uavsar.csv_final']),
  },
  mounted() {

    //create layers
    this.globalMap = new L.map('map').setView([36.9915, -119.7889], 5);
    L.control.scale({
      position: 'bottomright',
    }).addTo(this.globalMap);
    // load basemap
    //this.tileLayer();
    this.basemapLayers();

    var legend2 = L.control({position: 'bottomleft'});
    legend2.onAdd = function () {
      var div = L.DomUtil.create('div', 'info legend2');
      div.setAttribute("style","padding-left:60px;");
      div.innerHTML = '<img src="https://raw.githubusercontent.com/GeoGateway/geogateway-portal/master/html/images/logos/logo_black.png" style="height: 30px; width: 82px">';
      return div;
    };

    legend2.addTo(this.globalMap);

    var drawnItems = new L.FeatureGroup();
    this.globalMap.addLayer(drawnItems);

    this.drawControl = new L.Control.Draw({
      draw: {
        polygon: false,
        marker: false,
        polyline: false,
        circle: false,
        rectangle: false,
        circlemarker: false,
      },
    });
    this.globalMap.addControl(this.drawControl);

    bus.$on('UrlAddLayer', (url, layerName) =>
        this.kmlUrl(url, layerName));

    bus.$on('TextAddLayer', (text, layerName) =>
        this.kmlText(text, layerName));

    bus.$on('addExisting', (layerName) =>
        this.globalMap.addLayer(this.layers[layerName]));

    bus.$on('RemoveLayer', (name) =>
        this.globalMap.removeLayer(this.layers[name]));

    bus.$on('nowcast', (data, lat, lon) =>
        this.seismicityPlots(data, lat, lon));

    bus.$on('filterCat', (text, dFilter, mFilter, iconScale, startDate, endDate) =>
        this.catalogFilter(text, dFilter, mFilter, iconScale, startDate, endDate));

    bus.$on('gdacsGeoJSON', (text) =>
        this.addGdacsLayers(text));

    bus.$on('saveMapState', () =>
        this.saveState());

    bus.$on('displaySave', (layers) =>
        this.displaySave(layers));
    bus.$on('hidePlot', ()=> {
      bus.$emit('resetPlot');
    });
    bus.$on('clearSaveLayer', (layers) =>
        this.clearSave(layers));
    bus.$on('removeUavsarLayer', (name) =>
        this.removeUavsarLayer(name));
    bus.$on('activatePlot', (csv_final) =>
        this.showPlot(csv_final));
    bus.$on('showPlotDiv', () =>
        this.showPlotDiv());
    bus.$on('placePlotMarkers', (southwest, northeast, clickloc, latlon, entry) =>
        this.placePlotMarkers(southwest, northeast, clickloc, latlon, entry));
    bus.$on('RemovePlotPtGnss', () =>
        this.removePlotGnss());
    bus.$on('ClearUsgs', () =>
        this.clearUsgsLayers());
    bus.$on('addkmlUploadLayer', (file, filename) =>
        this.addkmlUploadLayer(file, filename));
    bus.$on('addGnssLayer', (file, type, prefix) =>
        this.addGnssLayer(file, type, prefix));
    bus.$on('seisDraw', () =>
        this.seismicityDraw());
    bus.$on('drawListenerOff', () =>
        this.globalMap.off('draw:created'));
    bus.$on('gnssDraw', () =>
        this.gnssDraw());
  },


  methods: {
    removeLayer(layerName) {
      this.globalMap.removeLayer(this.layers[layerName])
    },


    seismicityDraw(){
      new L.Draw.Rectangle(this.globalMap, this.drawControl.options.rectangle).enable();

      this.drawListener('seismicity');
    },
    gnssDraw(){
      new L.Draw.Rectangle(this.globalMap, this.drawControl.options.rectangle).enable();

      this.drawListener('gnss');
    },
    drawListener(tool){
      this.globalMap.on('draw:created', function (e) {
        var type = e.layerType;
        if (type === 'rectangle') {
          var layer = e.layer;
          this.addLayer(layer);
          this.centerLat = layer.getCenter().lat;
          this.centerLng = layer.getCenter().lng;
          this.maxLat = layer.getLatLngs()[0][1].lat;
          this.maxLon = layer.getLatLngs()[0][2].lng;
          this.minLat = layer.getLatLngs()[0][3].lat;
          this.minLon = layer.getLatLngs()[0][0].lng;
          this.removeLayer(layer)


          if(tool === 'uavsar'){
            bus.$emit('uavsarDrawQuery', this.maxLat, this.minLon, this.minLat, this.maxLon, this.centerLat, this.centerLng);
          }else if(tool === 'seismicity'){
            bus.$emit('seisDrawQuery', this.maxLat, this.minLon, this.minLat, this.maxLon, this.centerLat, this.centerLng);
          }else if(tool === 'gnss'){
            bus.$emit('gnssDrawQuery', this.maxLat, this.minLon, this.minLat, this.maxLon, this.centerLat, this.centerLng);
          }

          //control which tool hears bus event for drawing rect
        } else if (type === 'marker') {
          this.markerLayer = e.layer;
          var lat = this.markerLayer.getLatLng().lat;
          var lng = this.markerLayer.getLatLng().lng;
          bus.$emit('markPlace', lat, lng, tool);
        } else if (type === 'polygon') {
          var placedPolygon = e.layer;
          var arrLatLon = placedPolygon.getLatLngs();
          bus.$emit('polyDrawn', arrLatLon);
        }
      });

    },
    clearUsgsLayers() {
      this.layers['usgs_layer'].remove();
      this.layers['usgs_layer'] = null;
      this.usgsLegend.remove();
      this.usgsLegend = null;
    },
    showPlot(csv_final) {
      this.losPlot = new Dygraph(
          document.getElementById("dygraph-LOS"),
          csv_final, {
            labels:['distance','grc','dem'],
            series:{'grc':{axis:'y',drawPoints: true,pointSize: 2,strokeWidth: 0.0,showInRangeSelector: true},
              'dem':{axis:'y2',strokeWidth:1.0,drawPoints:false,showInRangeSelector: false},
              },
            // drawPoints: true,
            // pointSize: 2,
            // strokeWidth: 0.0,
            // titleHeight: 20,
            // xLabelHeight: 16,
            // yLabelWidth: 16, 
            xlabel: 'Distance (km)',
            ylabel: 'Ground Range Change (cm)',
            y2label: 'Ground Elevation (m) (line)',
            //maxNumberWidth: 5,
            sigFigs: 2,
            digitsAfterDecimal:2,
            //legend: 'always',
            showRangeSelector: true,
            axes: {
              y: {
              valueFormatter: y => y.toFixed(3),
              ticker: Dygraph.numericTicks,
              axisLabelFormatter: y => y.toFixed(1),
              pixelsPerLabel: 15,	      
              },
              y2: {
              valueFormatter: y => y.toFixed(3),
              ticker: Dygraph.numericTicks,
              axisLabelFormatter: y => y.toFixed(1),
              axisLabelWidth: 75,
              pixelsPerLabel: 15,
              },
              x: {
              valueFormatter: x => x.toFixed(3),
              ticker: Dygraph.numericTicks,
              axisLabelFormatter: x => x.toFixed(1),
              pixelsPerLabel: 35,
              }
            }
          }
      )
    },
    resizeLOS(e){
      if(this.losPlot != null) {
        let width = e.detail.width - 150;
        let height = e.detail.height - 110;
        // this.losPlot.updateOptions({'width': width - 30, 'height': height - 5})
        this.losPlot.resize(width, height);
      }

    },
    addkmlUploadLayer(file, filename) {
      function getExtension(f) {
        var parts = f.split('.');
        return parts[parts.length - 1];
      }

      var extension = getExtension(filename);
      if (extension == 'kmz') {
        this.layers[filename] = L.kmzLayer(file);
        this.globalMap.addLayer(this.layers[filename]);
      } else {
        this.kmlUrl(file, filename);
      }
    },

    //sets uavsar to overview mode

    showPlotDiv() {
      this.plotActive = true;
      this.$forceUpdate();
    },
    saveState() {
      bus.$emit('saved', this.layers);
    },
    displaySave(layers) {
      for (var key in layers) {
        if (layers[key] !== null) {
          this.globalMap.addLayer(layers[key])
        }
      }
    },
    //UAVSAR plot tile layer (for use with dygraphs)
    clearSave(layers) {
      for (var key in layers) {
        if (layers[key] !== null) {
          this.globalMap.removeLayer(layers[key]);
        }
      }
    },
    removeUavsarLayer(name) {
      this.globalMap.removeLayer(this.uavsarLayers[name]);
    },
    reactivateUavsarLayer(name) {
      this.globalMap.addLayer(this.uavsarLayers[name]);
    },

    // load basemap layer
    basemapLayers() { 

      var  tileProviders = [
        {
          name: 'ArcGIS Topo Map',
          visible: true,
          url: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Topo_Map/MapServer/tile/{z}/{y}/{x}',
          attribution:
            'Map data: &copy; <a href=<a href="http://www.esri.com/">Esri</a>',
          token:'',
          zIndex: 1 
        },
        {
          name: 'ArcGIS Light Map',
          visible: false,
          attribution:
            'Map data: &copy; <a href=<a href="http://www.esri.com/">Esri</a>',
          url: 'https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/{z}/{y}/{x}',
          token:'',
          zIndex: 2
         },
        {
          name: 'ArcGIS Dark Map',
          visible: false,
          attribution:
            'Map data: &copy; <a href=<a href="http://www.esri.com/">Esri</a>',
          url: 'https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Dark_Gray_Base/MapServer/tile/{z}/{y}/{x}',
          token:'',
          zIndex:3
        }
      ];

      var basemaps = {};
      var baseLayer;
      for (var tile of tileProviders) {
        baseLayer = L.tileLayer(tile.url,{attribution:tile.attribution});
        basemaps[tile['name']]=baseLayer;
        if (tile.visible == true) {baseLayer.addTo(this.globalMap);}
      }
      L.control.layers(basemaps).addTo(this.globalMap);
    },

    // tileLayer() {
    //   L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Topo_Map/MapServer/tile/{z}/{y}/{x}', {
    //     attribution: 'ArcGIS World Topo Map',zIndex:1
    //   }).addTo(this.globalMap);
    // },

    kmlText(text, layerName) {
      const parser = new DOMParser();
      const kml = parser.parseFromString(text, 'text/xml');
      this.layers[layerName] = new L.KML(kml);
      this.globalMap.addLayer(this.layers[layerName]);

    },
    addGnssLayer(file, type, prefix) {
      this.kmlText(file, type.concat(prefix));
    },
    removePlotGnss() {
      console.log(this.layers['gnssPlotPt'].remove());
    },
    kmlUrl(url, layerName) {
      fetch(url).then(res => res.text())
          .then(kmltext => {
            const parser = new DOMParser();
            var kml = parser.parseFromString(kmltext, "text/xml");
            this.layers[layerName] = new L.KML(kml);
            this.globalMap.addLayer(this.layers[layerName]);
          });

    },



    seismicityPlots(data, lat, lon) {
      var style = " style=width:200px;height:150px;"
      //fa-line-chart
      var latlng = L.latLng(lat, lon);
      this.layers['nowcastLayer'] = L.circleMarker(latlng, {
        color: 'green',
        fillColor: 'lightgreen',
        radius: 10,
      }).addTo(this.globalMap);

      var eps = data.urls[0]
      var numMag = data.urls[1]
      var seis = data.urls[2]

      //TODO make images clickable

      this.layers['nowcastLayer'].bindPopup("<img src=" + eps + style + " >" + "<br/>"
          + "<img src=" + numMag + style + " >"
          + "<img src=" + seis + style + " >");

      this.globalMap.panTo(new L.latLng(lat, lon));
    },
    catalogFilter(text, dFilter, mFilter, iconScale, startDate, endDate) {
      var toFilterM;
      if (this.layers['usgs_layer'] != null) {
        this.globalMap.removeLayer(this.layers['usgs_layer'])
      }
      //Better way to do this?
      //To add depth filter (what feature property represents depth?)
      if (mFilter === '') {
        toFilterM = () => true
      } else {
        toFilterM = function (feature) {
          return feature.properties.mag > parseInt(mFilter);
        }
      }
      this.layers['usgs_layer'] = L.geoJSON(text, {
        onEachFeature: function (feature, layer) {
          //what properties of each feature are most important to display?
          popupMaker(feature, layer);
        },
        filter: toFilterM,
        pointToLayer: function (feature, layer) {
          return circleMaker(feature, layer, iconScale, startDate, endDate);
        }
      }).addTo(this.globalMap)

      if (this.usgsLegend === null) {

        this.usgsLegend = L.control({position: 'bottomleft'});
        this.usgsLegend.onAdd = function () {
          var div = L.DomUtil.create('div', 'usgsLegend');
          div.innerHTML = "<b>"+startDate.toISOString().slice(0,10) +"</b> -- <b>"+endDate.toISOString().slice(0,10)+"</b>";
          div.innerHTML += "<br>"
          div.innerHTML += '<img  width="150" height="20" src="https://raw.githubusercontent.com/GeoGateway/GeoGatewayStaticResources/master/icons/rsz_gradient_linear.png">';
          return div;
        };

        this.usgsLegend.addTo(this.globalMap);
      }
    },

    addGdacsLayers(text) {
      this.layers['gdacsL'] = L.geoJSON(text, {
        onEachFeature: function (feature, layer) {
          //what properties of each feature are most important to display?
          gdacsPopup(feature, layer);
        },
        pointToLayer: function (feature) {
          var gdacsIcon = L.icon({
            iconUrl: feature.properties.iconitemlink,
            iconSize: [30, 35],
            iconAnchor: [15, 35],
          });
          var lat = feature.properties.latitude;
          var lon = feature.properties.longitude;
          return L.marker([lat, lon], {icon: gdacsIcon})
        }
      }).addTo(this.globalMap)
    },
  }
}
</script>
<style scoped>
#map {

  position: relative;
  height: 100%;
  /*z-index: 30000;*/
  width: auto;
  margin-left: auto;
  margin-bottom: auto;
  /*float: right;*/

}

#map-window {
  position: inherit;
  height: calc(100% - 40px);
  /*z-index: 30000;*/
  width: auto;
  padding: 0;
  /*float: right;*/
}

.dygraph-legend {
  background: black !important;
}


.leaflet-draw-toolbar a {
  background-image: url('../assets/spritesheet.png');
  background-repeat: no-repeat;
  color: transparent !important;
}

/*.plot{*/
/*    position: absolute;*/
/*    height: 200px;*/
/*    width: 500px;*/
/*    z-index: 400;*/
/*}*/
#dygraph-LOS {
  /*position: absolute;*/
  /*left: 1px;*/
  /*right: 1px;*/
  /*top: 1px;*/
  /*bottom: 1px;*/
  /*min-width: 100%;*/
  /*max-height: 100%;*/

}

#plot-window {
  position: absolute;
  z-index: 1500;
  background-color: #ccffcc;
  /*opacity: .5;*/
  height: 350px;
  resize: both;
  overflow: auto;
  width: 725px;
  border-radius: 20px;
  border: solid #343a40;
  border-width: thick;
}

#losLegend {
  background-color: #222222;
}
.dygraph-axis-label-y{
  float: right !important;
}

</style>

