<template>
    <div>
        <v-row>
            <!--  Graph of current Pressure in relation of Time  -->
            <v-col class="col-12 col-md-8 pb-0">
                <panel
                card-class="current_pressure"
                :icon="mdiArrowCollapseVertical"
                :title="'Current Pressure'"
                :collapsible="false">
                <temp-chart />
                </panel>
            </v-col>

            <!--  Panel to make a new Measurement and store it  -->
            <v-col class="col-12 col-md-4 pb-0">
                <panel
                card-class="new_measurement"
                :icon="mdiPlus"
                :title="'New Measurement'"
                :collapsible="false">

                    <div class="grid1" style="margin-top: 25px;">
                        <v-select v-model="new_filament_manufacturer"
                        :items="cfg.manufacturers"
                        label="Manufacturer"
                        style="margin-left: 15px; margin-right: 15px;">
                        </v-select>

                        <v-select v-model="new_filament_type"
                        :items="cfg.types"
                        label="Type"
                        style="margin-left: 15px; margin-right: 15px;">
                        </v-select>
                    </div>
                    <div class="grid2" style="justify-content: center; align-items: center; margin-bottom: 25px;">
                        <v-btn
                        icon
                        @click="new_filament_testtemp--"
                        :color="'primary'"
                        style="margin-left: 15px;">
                            <v-icon>{{ mdiChevronDown }}</v-icon>
                        </v-btn>

                        <v-text-field
                        v-model="new_filament_testtemp"
                        label="Temperature"
                        type="number"
                        hide-spin-buttons
                        style="margin-left: 10px; margin-right: 10px;"
                        ></v-text-field>

                        <v-btn
                        icon
                        @click="new_filament_testtemp++"
                        :color="'primary'"
                        style="margin-right: 15px;">
                            <v-icon>{{ mdiChevronUp }}</v-icon>
                        </v-btn>

                        <v-select v-model="new_filament_diameter"
                        :items="cfg.diameter"
                        label="Diameter"
                        style="margin-left: 15px; margin-right: 15px;">
                        </v-select>
                    </div>

                    <v-btn
                    block
                    :color="'primary'"
                    :disabled="isDisabled"
                    v-on:click="takeMeasurement(true)">Take Measurement</v-btn>
                </panel>
            </v-col>
        </v-row>
        <v-row>
            <!--  Panel to select stored Measurements  -->
            <v-col class="col-12 col-md-4 pb-0">
                <panel
                card-class="stored_measurements"
                :icon="mdiHistory"
                :title="'Stored Measurements'"
                :collapsible="false">
                    <div class="grid3" style="margin-top: 25px;">
                        <v-select v-model="filament_manufacturer"
                        :items="cfg.manufacturers"
                        label="Manufacturer"
                        style="margin-left: 15px; margin-right: 15px;">
                        </v-select>

                        <v-select v-model="filament_type"
                        :items="cfg.types"
                        label="Type"
                        style="margin-left: 15px; margin-right: 15px;">
                        </v-select>

                        <v-select v-model="filament_diameter"
                        :items="cfg.diameter"
                        label="Diameter"
                        style="margin-left: 15px; margin-right: 15px;">
                        </v-select>

                        <v-btn
                        style="margin-left: 15px; margin-right: 15px;"
                        color="primary"
                        @click="resetFilter()">Reset</v-btn>
                    </div>
                    <v-list lines="one" v-if="measurementsReady()">
                        <v-list-item v-for="measurement in filterMeasurements()" :key="measurement.id">
                            <v-list-item-content>
                                <v-list-item-title>
                                    {{ measurement.manufacturer + ' ' + measurement.type }}
                                </v-list-item-title>
                                <v-list-item-subtitle>
                                    Diameter: {{ measurement.diameter }}, Temperature: {{ measurement.temperature }} °C
                                </v-list-item-subtitle>
                            </v-list-item-content>
                            <v-btn :color="selectedMeasurement === measurement.id ? 'primary' : ''" @click="() => {
                            if(selectedMeasurement != measurement.id)
                                selectedMeasurement = measurement.id
                            else
                                selectedMeasurement = -1
                            if(selectedMeasurement != -1)
                                setChartOptions()
                            }">{{ selectedMeasurement === measurement.id ? 'SELECTED' : 'SELECT' }}</v-btn>
                            <v-btn icon @click="() => {deleteDialog = true; deleteSelected = measurement.id}" :color="'primary'" style="margin-left: 10px;">
                                <v-icon>{{ mdiDelete }}</v-icon>
                            </v-btn>
                        </v-list-item>
                    </v-list>
                </panel>
            </v-col>

            <!--  Show Graph of the selected stored Measurement  -->
            <v-col class="col-12 col-md-8">
                <panel
                card-class="graph"
                :icon="mdiChartAreaspline"
                :title="selectedMeasurement == -1 ? 'Graph' : measurements[selectedMeasurement].manufacturer + ' ' + measurements[selectedMeasurement].type + ' / ' + measurements[selectedMeasurement].diameter + ', ' + measurements[selectedMeasurement].temperature + ' °C'"
                :collapsible="false">
                <e-chart v-if="selectedMeasurement != -1"
                    ref="tempchart"
                    :option="chartOptions"
                    :autoresize="true"
                    style="height: 300px;"
                    class="tempchart" />
                <div v-else class="v-card__text ">Please select a Measurement to Display the Graph</div>
                </panel>
            </v-col>
        </v-row>
        <v-dialog v-model="deleteDialog" max-width="400">
            <panel
                :title="'Delete Measurement'"
                card-class="maschine-configfiles-delete-dialog"
                :margin-bottom="false">
                <template #buttons>
                    <v-btn icon tile @click="deleteDialog = false">
                        <v-icon>{{ mdiCloseThick }}</v-icon>
                    </v-btn>
                </template>
                <v-card-text>
                    <p class="mb-0">Do you really want to delete the selected Measurement?</p>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="" text @click="deleteDialog = false">Cancel</v-btn>
                    <v-btn color="error" text @click="() => {deleteMeasurement(); deleteDialog = false}">Delete</v-btn>
                </v-card-actions>
            </panel>
        </v-dialog>
        <v-dialog v-model="confirmDialog" max-width="400">
            <panel
                :title="'Take Measurement'"
                card-class="maschine-configfiles-delete-dialog"
                :margin-bottom="false">
                <template #buttons>
                    <v-btn icon tile @click="confirmDialog = false">
                        <v-icon>{{ mdiCloseThick }}</v-icon>
                    </v-btn>
                </template>
                <v-card-text>
                    <p class="mb-0">The Temperature is out of range. Do you want to proceed?</p>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="" text @click="confirmDialog = false">Cancel</v-btn>
                    <v-btn color="primary" text @click="() => {takeMeasurement(false); confirmDialog = false}">Proceed</v-btn>
                </v-card-actions>
            </panel>
        </v-dialog>
    </div>
</template>

<script lang="ts">
import BaseMixin from '@/components/mixins/base';
import Component from 'vue-class-component';
import { Mixins } from 'vue-property-decorator';
import { mdiSvg, mdiArrowCollapseVertical, mdiHistory, mdiPlus, mdiChartAreaspline, mdiChevronUp, mdiChevronDown, mdiDelete, mdiCloseThick } from '@mdi/js';

// Config-Interface where the Items displayed in the App are stored
interface Config{
    types: string[]
    min_temp: {[key: string]: number}
    max_temp: {[key: string]: number}
    diameter: string[]
    manufacturers: string[]
}

// Measurements-Interface where stored Measuremets are store in
interface Measurement{
    id: number
    manufacturer: string
    type: string
    diameter: string
    temperature: number
    flow: number[]
    pressure: number[]
}

@Component({
    components:{mdiSvg}
})

export default class PagePressure extends Mixins(BaseMixin) {
    mdiArrowCollapseVertical = mdiArrowCollapseVertical;
    mdiHistory = mdiHistory
    mdiPlus = mdiPlus
    mdiChartAreaspline = mdiChartAreaspline
    mdiChevronUp = mdiChevronUp
    mdiChevronDown = mdiChevronDown
    mdiDelete = mdiDelete
    mdiCloseThick = mdiCloseThick

    new_filament_type = ""
    new_filament_diameter = "1.75 mm"
    new_filament_manufacturer = ""
    new_filament_testtemp = 200

    filament_type = ""
    filament_manufacturer = ""
    filament_diameter= ""

    cfg: Config = this.getConfig()
    measurements: Measurement[] = this.getMeasurements()
    selectedMeasurement = -1
    deleteDialog = false
    confirmDialog = false
    deleteSelected = -1
    chartOptions!: Object

    // Checks if the Button to take new measurement is enabled or not
    get isDisabled():boolean{
        return !(this.new_filament_diameter && this.new_filament_type && this.new_filament_manufacturer && this.new_filament_testtemp);
    }

    // Loads the config from the filament.json-File and puts it into the Config-Interface
    async getConfig(): Promise<Config>{
        let ret = fetch('http://localhost:7125/server/files/config/addons/filament.json').then(response => response.json()).then(json => {
            const result: Config = {
                types: json.types,
                min_temp: {},
                max_temp: {},
                diameter: json.diameters,
                manufacturers: json.manufacturers,
            };

            json.types.forEach((type: string) => {
            const minTemp = json[type][0];
            const maxTemp = json[type][1];

            result.min_temp[type] = minTemp;
            result.max_temp[type] = maxTemp;
            });
            this.cfg = result
            return result;
        })
        return ret;
    }

    // Loads the Measurements from the measurements.json-File and puts them into the Measurements-Array
    async getMeasurements(): Promise<Measurement[]>{
        let ret = fetch('http://localhost:7125/server/files/config/addons/measurements.json').then(response => response.json()).then(json => {
            let id = 0
            let m = json.measurements.map((measurementData: any) => ({
                id: id++,
                manufacturer: measurementData.manufacturer,
                type: measurementData.type,
                diameter: measurementData.diameter,
                temperature: parseInt(measurementData.temperature),
                flow: measurementData.flow.map((flow: any) => parseFloat(flow)),
                pressure: measurementData.pressure.map((pressure: any) => parseFloat(pressure)),
            }));
            this.measurements = m;
            return m;
        })
        return ret;
    }

    // Returns the filtered Measurement-Array
    filterMeasurements(): Measurement[] {
        return this.measurements.filter((measurement: Measurement) => {
            const manufacturerFilter = !this.filament_manufacturer || measurement.manufacturer.toLowerCase() === this.filament_manufacturer.toLowerCase();
            const typeFilter = !this.filament_type || measurement.type.toLowerCase() === this.filament_type.toLowerCase();
            const diameterFilter = !this.filament_diameter || measurement.diameter.toLowerCase() === this.filament_diameter.toLowerCase();

            return manufacturerFilter && typeFilter && diameterFilter;
        });
    }

    resetFilter(){
        this.filament_type = ""
        this.filament_manufacturer = ""
        this.filament_diameter = ""
    }


    // Takes new Measurement and adds it to the stored Measurements
    takeMeasurement(confirm: boolean){
        let take = false;
        if(confirm && (this.new_filament_testtemp > this.cfg.max_temp[this.new_filament_type] || this.new_filament_testtemp < this.cfg.min_temp[this.new_filament_type]))
                this.confirmDialog = true
        else
            take = true;

        if(take == true)
            console.log("Measurement taken")
    }

    measurementsReady(): boolean{
        if(this.measurements instanceof Promise)
            return false;
        return true;
    }

    deleteMeasurement(){
        console.log("Delete Measurement " + this.deleteSelected)
        let pos = -1
        let i = -1
        this.measurements.forEach((m: Measurement) => {
            i++
            if(m.id == this.deleteSelected)
                pos = i
        })
        this.measurements.splice(pos, 1)
    }

    setChartOptions(){
        let pos = -1
        let i = -1
        this.measurements.forEach((m: Measurement) =>{
            i++
            if(m.id == this.selectedMeasurement)
                pos = i
        })

        const m = this.measurements[pos]

        let o: Object = {
            darkMode: true,
            animation: false,
            xAxis: {
                type: 'value',
                splitLine: {
                    show: true,
                    lineStyle: {
                        color: 'rgba(255, 255, 255, 0.06)',
                },
            },
            },
            yAxis: {
                type: 'value',
                splitLine: {
                    show: true,
                    lineStyle: {
                        color: 'rgba(255, 255, 255, 0.06)',
                },
            },
            },
            grid: {
                top: 35,
                right: 25,
                bottom: 30,
                left: 35
            },
            series: [
                {
                    name: 'flow - pressure',
                    type: 'line',
                    smooth: true,
                    data: m.flow.map((value, index) => [value, m.pressure[index]])
                }
            ],
        }

        this.chartOptions = o
    }
}
</script>

<style scoped>
.grid1 {
    display: grid;
    grid-template-columns: 1.6fr 1fr;
    grid-template-rows: 1fr;
    grid-column-gap: 0px;
    grid-row-gap: 0px;
}

.grid2 {
    display: grid;
    grid-template-columns: .01fr .5fr .01fr 1fr;
    grid-template-rows: 1fr;
    grid-column-gap: 0px;
    grid-row-gap: 0px;
}

.grid3 {
    display: grid;
    grid-template-columns: 1.5fr .8fr 1fr .5fr;
    grid-template-rows: 1fr;
    grid-column-gap: 0px;
    grid-row-gap: 0px;
}
</style>
