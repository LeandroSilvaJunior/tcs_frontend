<template>
    <el-row>
        <el-container>
            <el-header style="padding: 20px; background: mediumseagreen; font-size: 25px; border-bottom-style: solid ">
                <i class="el-icon-location"></i>
                GeoAirport
            </el-header>
        </el-container>
        <el-row>
            <el-col :span="24" style="padding: 24px">
                <l-map
                        style="height: 500px;"
                        :zoom="zoom"
                        :center="center"
                        :options="mapOptions"
                        @update:center="centerUpdate"
                        @update:zoom="zoomUpdate"
                >
                    <l-tile-layer
                            :url="url"
                            :subdomains="subdomains"
                    />
                    <l-marker
                            v-for="marker in markers"
                            :key="marker.id"
                            :visible="marker.visible"
                            :draggable="marker.draggable"
                            :lat-lng="marker.position"
                            :icon="icon"
                    >
                        <l-tooltip :content="marker.tooltip" />
                    </l-marker>
                    <l-polygon
                        :lat-lngs="polygon.latlngs"
                        :color="polygon.color"
                        :visible="showAreaProtegida"
                    />
                    <l-control position="bottomleft" >
                        <el-button @click="showPolygon()">
                            Alterar Visualização
                        </el-button>
                    </l-control>
                </l-map>
            </el-col>
        </el-row>
    </el-row>
</template>

<script>
    import {latLng, icon} from "leaflet";
    import { LMap, LTileLayer, LMarker, LTooltip, LPolygon, LControl} from "vue2-leaflet";

    const io = require('socket.io-client');

    export default {
        name: "Monitoramento",
        components: {
            LMap,
            LTileLayer,
            LMarker,
            LTooltip,
            LPolygon,
            LControl
        },
        data() {
            return {
                socket : io('http://localhost:3000'),
                zoom: 20,
                center: latLng(-26.87944, -48.64972),
                url: 'http://{s}.google.com/vt/lyrs=s&x={x}&y={y}&z={z}', //http://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}
                subdomains: ['mt0','mt1','mt2','mt3'],
                mapOptions: {
                    zoomSnap: 0.5
                },
                icon: icon({
                    iconUrl: require('@/assets/car.png'),
                    iconSize: [32, 37],
                    iconAnchor: [16, 37]
                }),
                iconSize: 64,
                markers: [],
                popupMessage: "",
                polygon: {
                    latlngs: [
                        [-26.883454, -48.659007],
                        [-26.884596, -48.658126],
                        [-26.880704, -48.651732],
                        [-26.881126, -48.651422],
                        [-26.878721, -48.647482],
                        [-26.878303, -48.647790],
                        [-26.875426, -48.642940],
                        [-26.874267, -48.643807]
                    ],
                    color: "red"
                },
                showAreaProtegida: true
            };
        },
        methods: {
            zoomUpdate(zoom) {
                this.zoom = zoom;
            },
            centerUpdate(center) {
                this.center = center;
            },
            rowCLick(column) {
                this.center = latLng(column.position.lat, column.position.lng);
                this.zoom = 20;
            },
            showPolygon() {
                this.showAreaProtegida = this.showAreaProtegida == true ? false : true;
            }
        },
        mounted() {
            this.socket.on('connect', function(){ console.log('websocket running!')});

            this.socket.on('updatedPosition', (markers) => {
                this.markers = markers;
            })
        }
    };
</script>