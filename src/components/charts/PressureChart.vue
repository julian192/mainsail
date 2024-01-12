<template>
    <e-chart
        ref="tempchart"
        v-observe-visibility="visibilityChanged"
        :option="chartOptions"
        :init-options="{ renderer: 'svg' }"
        :autoresize="true"
        :style="tempchartStyle"
        class="tempchart" />
</template>

<script lang="ts">
import Component from 'vue-class-component';
import { Mixins } from 'vue-property-decorator';
import BaseMixin from '../mixins/base';

@Component({
    components: {},
})

export default class PressureChart extends Mixins(BaseMixin){
    private isVisible = true;

    chartOptions: object = {
        darkMode: true,
        renderer: 'svg',
        animation: false,
        xAxis: {
            type: 'time',
            splitNumber: 5,
            minInterval: 60 * 1000,
            splitLine: {
                show: true,
                lineStyle: {
                    color: 'rgba(255, 255, 255, 0.06)',
                },
            },
            axisLabel: {
                color: 'rgba(255, 255, 255, 0.24)',
                margin: 10,
                formatter: this.timeFormat,
            },
        },
        yAxis: {
            type: 'value',
        },
        series: [{
            data: [120, 200, 150, 80, 70],
            type: 'line',
        }],
        grid: {
            top: 35,
            right: 25,
            bottom: 30,
            left: 35,
        },
    };

    tempchartStyle: object = {
        width: '100%',
        height: '300px',
    };    

    get timeFormat() {
        return this.hours12Format ? '{hh}:{mm}' : '{HH}:{mm}'
    }

    get series() {
        return this.$store.state.printer.tempHistory.series ?? {}
    }

    visibilityChanged(isVisible: boolean) {
        this.isVisible = isVisible
    }
}
</script>

<style scoped>
.tempchart {
    width: 100%;
}
</style>
