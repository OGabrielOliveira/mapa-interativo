<template>
    <div>
        <div ref="map" class="map"></div>
    </div>
</template>

<script>
import 'ol/ol.css';
import { Map, View } from 'ol';
import ImageLayer from 'ol/layer/Image';
import Static from 'ol/source/ImageStatic.js';
import { getCenter } from 'ol/extent.js';
import Projection from 'ol/proj/Projection.js';
import vanaris from '../assets/maps/vanaris.png';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import Style from 'ol/style/Style';
import Icon from 'ol/style/Icon';
import Popup from 'ol-popup';
import PopupMapa from './PopupMapa.vue';
import { createApp } from 'vue';

export default {
    name: 'MapaBase',
    data() {
        return {
            map: null,
            extent: [],
            projection: null,
            pointers: [
                {
                    coordinates: [270, 487],
                    name: 'Ilha Quark',
                    detail: 'Possui uma planta capaz de recuperar mana',
                    type: 'magic_plant'
                },
                {
                    coordinates: [224, 355],
                    name: 'Casa da Tanith',
                    detail: 'Possui uma placa de pedra como entrada',
                    type: 'safe_house'
                },
                {
                    coordinates: [210, 358],
                    name: 'Floresta Tanith',
                    type: 'spirit_forest_off'
                },
                {
                    coordinates: [300, 362],
                    name: 'Floresta chalé',
                    type: 'spirit_forest_off'
                },
                {
                    coordinates: [241, 404],
                    name: 'Esqueleto Icree',
                    detail: 'Abriga uma tribo de uma especie desconhecida',
                    type: 'danger_zone'
                },
                {
                    coordinates: [356, 395],
                    name: 'Kaer e Cyrr',
                    type: 'safe_house'
                },
                {
                    coordinates: [342, 358],
                    name: 'Toca da Gibbo',
                    type: 'danger_zone'
                },
                {
                    coordinates: [336, 302],
                    name: 'Toca dos goblins',
                    type: 'warning_zone'
                },
                {
                    coordinates: [330, 302],
                    name: 'Masmorra dos goblins',
                    detail: 'Masmorra descoberta pelo Ellru, fica na toca de tesouro',
                    type: 'dungeon_unexplored'
                },
                {
                    coordinates: [273, 128],
                    name: 'Caverna inexplorada',
                    type: 'danger_zone'
                },
                {
                    coordinates: [223, 272],
                    name: 'Floresta de Icree',
                    type: 'spirit_forest_on',
                    detail: 'Floresta de origem do Noriaki'
                },
                {
                    coordinates: [691, 612],
                    name: 'Ilha dragavit: Longa',
                    type: 'spirit_forest_on',
                    detail: 'Floresta de origem da Yukako'
                },
                {
                    coordinates: [376, 142],
                    name: 'Coliseu de Icree',
                    type: 'interest_point',
                    detail: 'Coordenado por um doppelganger'
                },
                {
                    coordinates: [244, 201],
                    name: 'Entrada clandestina Icree',
                    detail: 'Entrada utilizada pelo Haise e Emma para entrar em Icree',
                    type: 'route'
                },
                {
                    coordinates: [97, 311],
                    name: 'Cemitério',
                    type: 'danger_zone'
                },
                {
                    coordinates: [275, 368],
                    name: 'Geodo',
                    detail: 'Local que abrigava o Koichi',
                    type: 'crystal_mine'
                },
                {
                    coordinates: [256, 343],
                    name: 'Ravina de icree',
                    type: 'danger_zone'
                },
                {
                    coordinates: [458, 304],
                    name: 'Passagem pra Saran',
                    detail: 'Passagem subterrania criada pelo Boldur',
                    type: 'route'
                }
            ]

        }
    },
    mounted() {
        // Definição da projeção do mapa
        this.extent = [0, 0, 1266, 785];
        this.projection = new Projection({
            code: 'xkcd-image',
            unit: 'pixels',
            extent: this.extent
        });

        // Definição da camada de marcadores
        this.vector_source = new VectorSource({
            features: this.markers
        });

        let vector_layer = new VectorLayer({
            source: this.vector_source
        });

        // Inicialização do mapa
        this.initMap();
        this.map.addLayer(vector_layer);

        this.Popup = new Popup({
            popupClass: 'default anim',
            closeBox: true,
            positioning: 'top-center',
            autoPan: true,
            autoPanAnimation: {
                duration: 250
            }
        });

        this.map.addOverlay(this.Popup);

        // Adição dos marcadores
        for (let pointer of this.pointers) {
            let feature = this.create_pointer(pointer);
            vector_layer.getSource().addFeature(feature);
        }


        // Evento de click no mapa
        this.map.on('click', (evt) => {
            console.log(evt.coordinate);
            let feature = this.vector_source.getClosestFeatureToCoordinate(evt.coordinate);

            if (this.verificaDentroDoRaio(feature.values_.geometry.flatCoordinates, evt.coordinate)) {
                console.log(feature.values_);

                const app = createApp(PopupMapa, {
                    name: feature.values_.name.name,
                    detail: feature.values_.name.detail,
                });

                const instance = app.mount(document.createElement('div'));
                const popupContent = instance.$el;

                this.Popup.show(evt.coordinate, popupContent);
            } else {
                this.Popup.hide();
            }

            console.log(this.verificaDentroDoRaio(evt.coordinate, feature.values_.geometry.flatCoordinates));
            // console.log(feature.value_.geometry.flatCoordinates);
        });
    },
    methods: {
        verificaDentroDoRaio(click_cords, point_coords) {
            let x = click_cords[0] - point_coords[0];
            let y = click_cords[1] - point_coords[1];
            let distance = Math.sqrt(x * x + y * y);
            return distance <= 15;
        },
        create_pointer(pointer_data) {
            let feature = new Feature({
                type: 'icon',
                geometry: new Point(pointer_data.coordinates),
                name: { 'name': pointer_data.name, 'detail': pointer_data.detail }
            });

            feature.setStyle(new Style({
                image: new Icon({
                    src: require(`../assets/icons/${pointer_data.type}.png`),
                    scale: 0.8,
                    anchor: [0.5, 1],
                })
            }));

            return feature;
        },
        initMap() {
            this.map = new Map({
                target: this.$refs.map,
                layers: [
                    new ImageLayer({
                        source: new Static({
                            url: vanaris,
                            projection: this.projection,
                            imageExtent: this.extent,
                        })
                    })
                ],
                view: new View({
                    projection: this.projection,
                    center: getCenter(this.extent),
                    zoom: 2.5,
                    maxZoom: 8,
                })
            });
        }
    }
}
</script>

<style scoped>
.map {
    width: calc(100vw - 15px);
    height: calc(100vh - 15px);
    position: absolute;
    background-color: #468889;
}
</style>