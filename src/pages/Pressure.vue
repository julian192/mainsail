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
                    v-on:click="takeMeasurement">Take Measurement</v-btn>
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
                    </div>
                </panel>
            </v-col>

            <!--  Show Graph of the selected stored Measurement  -->
            <v-col class="col-12 col-md-8">
                <panel
                card-class="graph"
                :icon="mdiChartAreaspline"
                :title="'Graph'"
                :collapsible="false">
                <temp-chart />
                </panel>
            </v-col>
        </v-row>
    </div>
</template>

<script lang="ts">
import BaseMixin from '@/components/mixins/base';
import Component from 'vue-class-component';
import { Mixins } from 'vue-property-decorator';
import { mdiSvg, mdiArrowCollapseVertical, mdiHistory, mdiPlus, mdiChartAreaspline, mdiChevronUp, mdiChevronDown } from '@mdi/js';
import * as pp from 'path'

interface Config{
    types: string[]
    min_temp: {[key: string]: number}
    max_temp: {[key: string]: number}
    diameter: string[]
    manufacturers: string[]
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

    new_filament_type = ""
    new_filament_diameter = "1.75 mm"
    new_filament_manufacturer = ""
    new_filament_testtemp = 200

    filament_type = ""
    filament_manufacturer = ""
    filament_diameter= "1.75 mm"

    cfg: Config = this.getConfig()

    // Checks if the Button to take new measurement is enabled or not
    get isDisabled():boolean{
        return !(this.new_filament_diameter && this.new_filament_type && this.new_filament_manufacturer && this.new_filament_testtemp);
    }

    // Loads the config from the filament.json-File and puts it into the Config-Interface
    async getConfig(): Promise<Config>{
        let ret!: Config
        fetch('http://localhost:7125/server/files/config/addons/filament.json').then(response => response.json()).then(json => {
            const parsedData = json

            const result: Config = {
                types: parsedData.types,
                min_temp: {},
                max_temp: {},
                diameter: parsedData.diameters,
                manufacturers: parsedData.manufacturers,
            };

            parsedData.types.forEach((type: string) => {
            const minTemp = parsedData[type][0];
            const maxTemp = parsedData[type][1];

            result.min_temp[type] = minTemp;
            result.max_temp[type] = maxTemp;
            });
            this.cfg = result
            ret = result
            return result;
        })
        return ret;
    }

    // Takes new Measurement and adds it to the stored Measurements
    takeMeasurement(){
        let take = false;
        if(this.new_filament_testtemp > this.cfg.max_temp[this.new_filament_type] || this.new_filament_testtemp < this.cfg.min_temp[this.new_filament_type]){
            if(confirm("The Temperature is not in the typical range of the filament type \"" + this.new_filament_type + "\""))
                take = true
        }else{
            take = true;
        }
        if(take == true){
            console.log("Measurement taken")
        }
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
    grid-template-columns: 1.8fr 1fr .9fr;
    grid-template-rows: 1fr;
    grid-column-gap: 0px;
    grid-row-gap: 0px;
}
</style>