<template>
    <div id="container">
        <div id="info">
            <div v-if="this.attraction.name">
                <p>
                    <strong>Name:</strong>
                    {{this.attraction.name}}
                </p>
                <p>
                    <strong>Address:</strong>
                    {{this.attraction.address}}
                </p>
                <p>
                    <strong>Type:</strong>
                    {{this.attraction.category}}
                </p>
            </div>
        </div>
        <div id="mapContainer"></div>
    </div>
</template>

<script>
    import "leaflet/dist/leaflet.css";
    import L from "leaflet";
    import axios from "axios";
    // import data from "./Historic-Landmarks.json";

    export default {
        name: "Map",
        data() {
            return {
                center: [10.762407, 106.681549],
                map: null,
                attraction: {
                    name: "",
                    address: "",
                    category: "",
                },
                clientSecret: "FTZGMLOIQWFY3A0ELEZIZSUU3M4EKOJKEPXKWUWTMWK1EY4H",
                clientID: "GOSFGAOZKCSLMWADY1ORYJV2A4GUNNHAHBVWY500S1IM42CS",
            };
        },
        methods: {
            fetchDormitories () {
                return axios.get(`https://api.luuxaconggiao.tk/web/public/dormitory/list`);
            },
            fetchUniversities () {
                return axios.get(`https://api.luuxaconggiao.tk/web/public/university/list`);
            },
            setupLeafletMap () {
                this.map = L.map("mapContainer").setView(this.center, 13);
                L.tileLayer(
                    "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}",
                    {
                        attribution: 'Map data © <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery (c) <a href="https://www.mapbox.com/">Mapbox</a>',
                        maxZoom: 18,
                        id: "mapbox/streets-v11",
                        accessToken: "pk.eyJ1IjoiYWJpZGlzaGFqaWEiLCJhIjoiY2l3aDFiMG96MDB4eDJva2l6czN3MDN0ZSJ9.p9SUzPUBrCbH7RQLZ4W4lQ",
                    }
                ).addTo(this.map);
                this.fetchDormitories().then(result => {
                    for (let idx in result.data) {
                        let dormitory = result.data[idx];
                        // L.circle([dormitory.lat, dormitory.lng], {radius: 200}).addTo(this.map);
                        let myIcon = L.icon({
                            // iconUrl: 'https://cdn0.iconfinder.com/data/icons/small-n-flat/24/678085-house-512.png',
                            iconUrl: dormitory.logo,
                            iconSize: [30, 30],
                            // iconAnchor: [22, 94],
                            // popupAnchor: [-3, -76],
                            // shadowUrl: 'my-icon-shadow.png',
                            // shadowSize: [68, 95],
                            // shadowAnchor: [22, 94]
                        });

                        L.marker([dormitory.lat, dormitory.lng], {icon: myIcon})
                            .addTo(this.map)
                            .bindPopup(dormitory.name);
                    }
                })
                this.fetchUniversities().then(result => {
                    for (let idx in result.data) {
                        let university = result.data[idx];
                        let myIcon = L.icon({
                            iconUrl: university.logo,
                            iconSize: [30, 30],
                            // iconAnchor: [22, 94],
                            // popupAnchor: [-3, -76],
                            // shadowUrl: 'my-icon-shadow.png',
                            // shadowSize: [68, 95],
                            // shadowAnchor: [22, 94]
                        });

                        L.marker([university.lat, university.lng], {icon: myIcon})
                            .addTo(this.map)
                            .on("click", this.onClick)
                            .bindPopup(university.name);
                    }
                })

                // this.map.on('zoomend', function() {
                //     if (this.map.getZoom() <7){
                //         this.map.removeLayer(shelterMarkers);
                //     }
                //     else {
                //         this.map.addLayer(shelterMarkers);
                //     }
                // });

                this.map.on('click', this.onMapClick);
            },
            onMapClick(e) {
                let popup = L.popup();
                popup
                    .setLatLng(e.latlng)
                    .setContent("You clicked the map at " + e.latlng.toString())
                    .openOn(this.map);
            },
            onClick(e) {
                let latlng = this.map.mouseEventToLatLng(e.originalEvent);
                let map = this.map.setView([latlng.lat, latlng.lng], 13);

                L.circle([latlng.lat, latlng.lng], 500, {
                    color: '#6de2fc',
                    fillColor: '#0f66f2',
                    fillOpacity: 0.1
                }).addTo(map);

                // let myZoom = {
                //     start:  map.getZoom(),
                //     end: map.getZoom()
                // };
                //
                // // eslint-disable-next-line no-unused-vars
                // map.on('zoomstart', function(_) {
                //     myZoom.start = map.getZoom();
                // });
                //
                // // eslint-disable-next-line no-unused-vars
                // map.on('zoomend', function(_) {
                //     myZoom.end = map.getZoom();
                //     var diff = myZoom.start - myZoom.end;
                //     if (diff > 0) {
                //         circle.setRadius(circle.getRadius() * 2);
                //     } else if (diff < 0) {
                //         circle.setRadius(circle.getRadius() / 2);
                //     }
                // });
            }
        },
        mounted() {
            this.setupLeafletMap();
        },
    };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h3 {
        margin: 40px 0 0;
    }
    ul {
        list-style-type: none;
        padding: 0;
    }
    li {
        display: inline-block;
        margin: 0 10px;
    }
    a {
        color: #42b983;
    }

    #container {
        display: flex;
    }
    #info {
        width: 18vw;
        height: 100vh;
    }
    #mapContainer {
        width: 80vw;
        height: 100vh;
    }
</style>
