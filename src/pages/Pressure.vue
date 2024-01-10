<template>
    <div v-if="klipperReadyForGui">
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
                        :items="filament_manufacturers"
                        label="Manufacturer"
                        style="margin-left: 15px; margin-right: 15px;">
                        </v-select>

                        <v-select v-model="new_filament_type"
                        :items="filament_types"
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
                        :items="filament_diameters"
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
                        :items="filament_manufacturers"
                        label="Manufacturer"
                        style="margin-left: 15px; margin-right: 15px;">
                        </v-select>

                        <v-select v-model="filament_type"
                        :items="filament_types"
                        label="Type"
                        style="margin-left: 15px; margin-right: 15px;">
                        </v-select>

                        <v-select v-model="filament_diameter"
                        :items="filament_diameters"
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
import ConfigFilesPanel from '@/components/panels/Machine/ConfigFilesPanel.vue';

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

    new_filament_type!: string
    new_filament_diameter!: string
    new_filament_manufacturer!: string
    new_filament_testtemp!: number

    filament_type!: string
    filament_manufacturer!: string
    filament_diameter!: string

    cfg: Config = this.getConfig()
    
    /**
    filament_types: string[] = ["PLA", "PLA+", "ABS", "ASA", "PETG", "TPU", "Nylon", "HIPS"];
    filament_min_temp: { [key: string]: number } = {"PLA": 180, "PLA+": 180, "ABS": 220, "ASA": 230, "PETG": 220, "TPU": 220, "Nylon": 220, "HIPS": 230}
    filament_max_temp: { [key: string]: number } = {"PLA": 220, "PLA+": 220, "ABS": 250, "ASA": 260, "PETG": 250, "TPU": 250, "Nylon": 270, "HIPS": 250}
    filament_diameters: string[] = ["1.75 mm"];
    filament_manufacturers: string[] = ["eSun", "Prusa", "Geeetech", "Polymaker", "Sunlu"];
    */

    // Checks if the Button to take new measurement is enabled or not
    get isDisabled():boolean{
        return !(this.new_filament_diameter && this.new_filament_type && this.new_filament_manufacturer && this.new_filament_testtemp);
    }

    // Loads the config from the pressure.cfg-File and puts it into the Config-Interface
    getConfig(): Config{

        return {types: [], min_temp: {}, max_temp: {}, diameter: [], manufacturers: []}; 
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