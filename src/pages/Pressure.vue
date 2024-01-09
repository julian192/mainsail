<template>
    <div v-if="klipperReadyForGui">
        <v-row>
            <!--  Graph of current Pressure in relation of Time  -->
            <v-col class="col-12 col-md-8 pb-0">
                <panel 
                :icon="mdiArrowCollapseVertical"
                :title="'Current Pressure'"
                :collapsible="false">
                <temp-chart />
                </panel>
            </v-col>

            <!--  Panel to make a new Measurement and store it  -->
            <v-col class="col-12 col-md-4 pb-0">
                <panel
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
                    :disabled="isDisabled">Add Measurement</v-btn>
                </panel>
            </v-col>
        </v-row>
        <v-row>
            <!--  Panel to select stored Measurements  -->
            <v-col class="col-12 col-md-4 pb-0">
                <panel
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
    new_filament_testtemp = 220

    filament_type = "PLA"
    filament_manufacturer = "Prusa"
    filament_diameter = "1.75 mm"

    filament_types: string[] = ["PLA", "PLA+", "ABS", "ASA", "PETG", "TPU"];
    filament_diameters: string[] = ["1.75 mm"];
    filament_manufacturers: string[] = ["E-Sun", "Prusa", "Geeetech"];

    get isDisabled():boolean{
        return !(this.new_filament_diameter && this.new_filament_type && this.new_filament_manufacturer && this.new_filament_testtemp);
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