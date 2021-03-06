<template>
    <div class="planet-picker-comp panel panel-default">
        <div class="panel-heading">
            Orbital Body
        </div>
        <div class="panel-body">
            <p>
                <button-group
                    :buttonLabels="celBodies"
                    :selected="selectedPlanetName"
                    v-on:selected="buttonClicked"
                    :class="buttonGroupClass">
                </button-group>
            </p>
            <p>
                <button-group
                    v-for="body in celBodies"
                    :selected="selectedSatName"
                    v-bind:key="body.name"
                    v-show="selectedPlanetName == body.name"
                    v-on:selected="satButtonClicked"
                    :buttonLabels="body.satellites"
                    :class="satButtonGroupClass">
                </button-group>
            </p>
            <planet-info :bodyInfo="selectedBody"></planet-info>
        </div>
    </div>
</template>

<style>
.sat-group {
    clear: both;
}
</style>

<script>
import _ from 'lodash'
import ButtonGroup from './ButtonGroup'
import PlanetInfo from './PlanetInfo'
const smallScreenThreshold = 512;

export default {
    name: 'PlanetPicker',
    components: {
        ButtonGroup,
        PlanetInfo
    },
    props: {
        celBodies: {
            type: Array,
            required: true
        },
        selectedPlanetIx: {
            type: Number,
            required: true
        },
        selectedSatIx: {
            default: null
        }
    },
    data () {
        return {
            isMounted: false,
            isSmall: window.innerWidth < smallScreenThreshold,
            smallScreenThreshold: smallScreenThreshold
        }
    },
    created: function () {
        var self = this;
        window.addEventListener('resize', function () {
            self.onResize();
        });
    },
    mounted: function () {
        this.isMounted = true;
    },
    computed: {
        selectedPlanetName: function () {
            return this.selectedPlanet.name;
        },
        selectedPlanet: function () {
            return this.celBodies[this.selectedPlanetIx];
        },
        selectedSatName: function () {
            var satBody = this.selectedSat;
            if (satBody) {
                return satBody.name;
            } else {
                return null;
            }
        },
        selectedSat: function () {
            return this.selectedPlanet.satellites[this.selectedSatIx];
        },
        selectedBody: function () {
            if (this.selectedSat) {
                return this.selectedSat;
            } else {
                return this.selectedPlanet;
            }
        },
        buttonGroupClass: function () {
            return {
                'btn-group-xs': this.isSmall,
                'btn-group-sm': !this.isSmall
            };
        },
        satButtonGroupClass: function () {
            return {
                'btn-group-xs': this.isSmall,
                'btn-group-sm': !this.isSmall,
                'sat-group': true
            };
        }
    },
    methods: {
        onResize: function () {
            this.isSmall = window.innerWidth < this.smallScreenThreshold;
        },
        getIndexFromName: function (name) {
            return _.findIndex(this.celBodies, {name: name});
        },
        getSatIndexFromName: function (name) {
            return _.findIndex(this.selectedPlanet.satellites, {name: name});
        },
        buttonClicked: function (name) {
            var selectedIndex = this.getIndexFromName(name);
            this.$emit('picked', {
                name: name,
                index: selectedIndex,
                satIndex: null
            });
        },
        satButtonClicked: function (name) {
            var planetName = this.selectedPlanetName;
            var selectedSatIndex = this.getSatIndexFromName(name);
            if (selectedSatIndex === this.selectedSatIx) {
                selectedSatIndex = null;
            }
            this.$emit('picked', {
                name: planetName,
                index: this.selectedPlanetIx,
                satIndex: selectedSatIndex
            });
        },
    }
}
</script>