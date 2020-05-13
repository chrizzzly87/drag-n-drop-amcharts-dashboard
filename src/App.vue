<template>
    <div id="app">
        <button type="button" @click="addGridItem">Add Grid Item</button>
        <grid-layout
            :layout.sync="layout"
            :col-num="4"
            :row-height="300"
            :is-draggable="true"
            :is-resizable="true"
            :is-mirrored="false"
            :vertical-compact="true"
            :margin="[10, 10]"
            :use-css-transforms="true"
        >
            <grid-item
                v-for="item in layout"
                :x="item.x"
                :y="item.y"
                :w="item.w"
                :h="item.h"
                :i="item.i"
                :key="item.i"
            >
                <div :id="item.i" class="item__wrapper">
                    <div class="item__header">
                        <h2 class="item__headline">Empty Headline</h2>
                        <p class="item__subline">
                            Target group:
                            <span class="chip chip--closeable">Male GenY</span>
                        </p>
                    </div>
                    <div class="item__content">Nothing inside</div>
                    <button
                        type="button"
                        style="z-index:1000"
                        @click="removeGridItem(item.i)"
                    >Remove</button>
                </div>
            </grid-item>
        </grid-layout>
    </div>
</template>

<script>
import VueGridLayout from "vue-grid-layout";
import * as am4core from "@amcharts/amcharts4/core";
import * as am4charts from "@amcharts/amcharts4/charts";
import am4themes_animated from "@amcharts/amcharts4/themes/animated";

function am4themes_gcs(target) {
    if (target instanceof am4core.ColorSet) {
        target.list = [
            am4core.color("#002F63"),
            am4core.color("#1883CD"),
            am4core.color("#1EB9E7"),
            am4core.color("#53DDC4"),
            am4core.color("#33C972"),
            am4core.color("#00A563")
        ];
    }
}

am4core.useTheme(am4themes_animated);
am4core.useTheme(am4themes_gcs);

let defaultLayout = [
    {
        x: 0,
        y: 0,
        w: 1,
        h: 1,
        i: "radialChart",
        maxH: 2
    },
    { x: 1, y: 0, w: 1, h: 1, i: "xychart", maxH: 2 },
    { x: 2, y: 0, w: 2, h: 1, i: "divergentchart", maxH: 2 }
];

export default {
    name: "App",
    components: {
        GridLayout: VueGridLayout.GridLayout,
        GridItem: VueGridLayout.GridItem
    },
    data: function() {
        return {
            layout: defaultLayout,
            customItemCount: 0
        };
    },
    methods: {
        removeGridItem(id) {
            this.layout = this.layout.filter(function(el) {
                return el.i != id;
            });
        },
        addGridItem() {
            this.customItemCount += 1;
            this.layout.push({
                x: 0,
                y: 0,
                w: 1,
                h: 1,
                i: `customItem${this.customItemCount}`
            });
        }
    },
    mounted() {
        createRadarChart();
        createDivergentStackedChart();
        createXYChart();
    },
    beforeDestroy() {
        if (this.chart) {
            this.chart.dispose();
        }
    }
};

function createRadarChart() {
    let chart = am4core.create("radialChart", am4charts.RadarChart);

    // Add data
    chart.data = [
        {
            category: "Students",
            value: 80,
            full: 100
        },
        {
            category: "Gen Y / Millenials",
            value: 35,
            full: 100
        },
        {
            category: "Brand Advocates",
            value: 92,
            full: 100
        },
        {
            category: "Parents",
            value: 68,
            full: 100
        }
    ];

    // Make chart not full circle
    chart.startAngle = -90;
    chart.endAngle = 180;
    chart.innerRadius = am4core.percent(20);

    // Set number format
    chart.numberFormatter.numberFormat = "#.#'%'";

    // Create axes
    var categoryAxis = chart.yAxes.push(new am4charts.CategoryAxis());
    categoryAxis.dataFields.category = "category";
    categoryAxis.renderer.grid.template.location = 0;
    categoryAxis.renderer.grid.template.strokeOpacity = 0;
    categoryAxis.renderer.labels.template.horizontalCenter = "right";
    categoryAxis.renderer.labels.template.fontWeight = 500;
    categoryAxis.renderer.labels.template.adapter.add("fill", function(
        fill,
        target
    ) {
        return target.dataItem.index >= 0
            ? chart.colors.getIndex(target.dataItem.index)
            : fill;
    });
    categoryAxis.renderer.minGridDistance = 20;

    var valueAxis = chart.xAxes.push(new am4charts.ValueAxis());
    valueAxis.renderer.labels.template.fontWeight = 500;
    valueAxis.renderer.labels.template.fontSize = "13px";
    valueAxis.renderer.labels.template.fill = "#566d82";
    valueAxis.renderer.grid.template.strokeOpacity = 0;
    valueAxis.min = 0;
    valueAxis.max = 100;
    valueAxis.strictMinMax = true;

    // Create series - the backdrop
    var series1 = chart.series.push(new am4charts.RadarColumnSeries());
    series1.dataFields.valueX = "full";
    series1.dataFields.categoryY = "category";
    series1.clustered = false;
    series1.columns.template.fill = new am4core.InterfaceColorSet().getFor(
        "alternativeBackground"
    );
    series1.columns.template.fillOpacity = 0.08;
    series1.columns.template.cornerRadiusTopLeft = 20;
    series1.columns.template.strokeWidth = 0;
    series1.columns.template.radarColumn.cornerRadius = 20;

    var series2 = chart.series.push(new am4charts.RadarColumnSeries());
    series2.dataFields.valueX = "value";
    series2.dataFields.categoryY = "category";
    series2.clustered = false;
    series2.columns.template.strokeWidth = 0;
    series2.columns.template.tooltipText = "{category}: [bold]{value}[/]";
    series2.columns.template.radarColumn.cornerRadius = 20;
    series2.sequencedInterpolation = true;

    series2.columns.template.adapter.add("fill", function(fill, target) {
        return chart.colors.getIndex(target.dataItem.index);
    });
}

function createDivergentStackedChart() {
    // Create chart instance
    var chart = am4core.create("divergentchart", am4charts.XYChart);

    // Title
    var title = chart.titles.push(new am4core.Label());
    title.text = "Movie genre popularity";
    title.fontSize = 20;
    title.marginBottom = 15;

    // Add data
    chart.data = [
        {
            category: "Comedy",
            negative1: -0.1,
            negative2: -0.9,
            positive1: 5,
            positive2: 94
        },
        {
            category: "Action",
            negative1: -2,
            negative2: -4,
            positive1: 19,
            positive2: 75
        },
        {
            category: "Thriller",
            negative1: -2,
            negative2: -10,
            positive1: 46,
            positive2: 42
        },
        {
            category: "Drama",
            negative1: -2,
            negative2: -13,
            positive1: 33,
            positive2: 52
        },
        {
            category: "Western",
            negative1: -6,
            negative2: -19,
            positive1: 34,
            positive2: 41
        },
        {
            category: "Black/White",
            negative1: -3,
            negative2: -23,
            positive1: 49,
            positive2: 25
        }
    ];

    // Create axes
    var categoryAxis = chart.yAxes.push(new am4charts.CategoryAxis());
    categoryAxis.dataFields.category = "category";
    categoryAxis.renderer.grid.template.location = 0;
    categoryAxis.renderer.inversed = true;
    categoryAxis.renderer.minGridDistance = 20;
    categoryAxis.renderer.axisFills.template.disabled = false;
    categoryAxis.renderer.axisFills.template.fillOpacity = 0.05;

    var valueAxis = chart.xAxes.push(new am4charts.ValueAxis());
    valueAxis.min = -100;
    valueAxis.max = 100;
    valueAxis.renderer.minGridDistance = 50;
    valueAxis.renderer.ticks.template.length = 5;
    valueAxis.renderer.ticks.template.disabled = false;
    valueAxis.renderer.ticks.template.strokeOpacity = 0.4;
    valueAxis.renderer.labels.template.adapter.add("text", function(text) {
        return text == "Male" || text == "Female" ? text : text + "%";
    });

    // Legend
    chart.legend = new am4charts.Legend();
    chart.legend.position = "right";

    // Use only absolute numbers
    chart.numberFormatter.numberFormat = "#.#s";

    // Create series
    function createSeries(field, name, color) {
        var series = chart.series.push(new am4charts.ColumnSeries());
        series.dataFields.valueX = field;
        series.dataFields.categoryY = "category";
        series.stacked = true;
        series.name = name;
        series.stroke = color;
        series.fill = color;
        series.sequencedInterpolation = true;

        var label = series.bullets.push(new am4charts.LabelBullet());
        label.label.text = "{valueX}%";
        label.label.fill = am4core.color("#fff");
        label.label.strokeWidth = 0;
        label.label.truncate = false;
        label.label.hideOversized = true;
        label.locationX = 0.5;
        return series;
    }

    var interfaceColors = new am4core.InterfaceColorSet();
    var positiveColor = interfaceColors.getFor("positive");
    var negativeColor = interfaceColors.getFor("negative");

    createSeries("negative2", "Unlikely", negativeColor.lighten(0.5));
    createSeries("negative1", "Never", negativeColor);
    createSeries("positive1", "Sometimes", positiveColor.lighten(0.5));
    createSeries("positive2", "Very often", positiveColor);

    chart.legend.events.on("layoutvalidated", function() {
        chart.legend.itemContainers.each(container => {
            if (container.dataItem.dataContext.name == "Never") {
                container.toBack();
            }
        });
    });
}

function createXYChart() {
    // Create chart instance
    var chart = am4core.create("xychart", am4charts.XYChart);

    // Add data
    chart.data = [
        {
            date: "2012-03-01",
            price: 20
        },
        {
            date: "2012-03-02",
            price: 75
        },
        {
            date: "2012-03-03",
            price: 15
        },
        {
            date: "2012-03-04",
            price: 75
        },
        {
            date: "2012-03-05",
            price: 158
        },
        {
            date: "2012-03-06",
            price: 57
        }
    ];

    // Create axes
    var dateAxis = chart.xAxes.push(new am4charts.DateAxis());
    dateAxis.renderer.grid.template.location = 0;
    dateAxis.renderer.minGridDistance = 50;
    dateAxis.renderer.labels.template.labelRotation = 45;

    var valueAxis = chart.yAxes.push(new am4charts.ValueAxis());
    valueAxis.renderer.minGridDistance = 50;

    // Create series
    var series = chart.series.push(new am4charts.LineSeries());
    series.dataFields.valueY = "price";
    series.dataFields.dateX = "date";
    series.tensionX = 0.8;
    series.strokeWidth = 3;
    series.tooltipText = "{value}";

    var bullet = series.bullets.push(new am4charts.CircleBullet());
    bullet.circle.fill = am4core.color("#fff");
    bullet.circle.strokeWidth = 3;

    // Add cursor
    // chart.cursor = new am4charts.XYCursor();
    // chart.cursor.fullWidthLineX = true;
    // chart.cursor.xAxis = dateAxis;
    // chart.cursor.lineX.strokeWidth = 0;
    // chart.cursor.lineX.fill = am4core.color("#000");
    // chart.cursor.lineX.fillOpacity = 0.1;

    // Add scrollbar
    // chart.scrollbarX = new am4core.Scrollbar();
}
</script>

<style>
#app {
    font-family: "Open Sans", Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;

    background-color: rgb(237, 242, 247);
    width: 100vw;
    height: 100vh;
}
.vue-grid-item {
    opacity: 1;
    background-color: #ffffff; /* #a6b6c4 */
    border-radius: 3px;

    /* Trick: Use pseudo element for better performance on box-shadows! */
    position: relative;
    box-shadow: 0 0.2px 1.4px -9px rgba(0, 0, 0, 0.02),
        0 0.5px 3.3px -9px rgba(0, 0, 0, 0.028),
        0 0.9px 6.1px -9px rgba(0, 0, 0, 0.035),
        0 1.6px 10.9px -9px rgba(0, 0, 0, 0.042),
        0 2.9px 20.5px -9px rgba(0, 0, 0, 0.05),
        0 7px 49px -9px rgba(0, 0, 0, 0.07);
}

/* element when resized */
.vue-grid-item.resizing {
    opacity: 1;
    background-color: rgba(255, 255, 255, 0.3);
}
/* dragging */
/* foreground element */
.vue-grid-item.vue-draggable-dragging {
    opacity: 1;
    background-color: #fff;
}
.vue-grid-item::before {
    content: " ";
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    opacity: 0;
    transition: opacity 500ms;

    box-shadow: 0 2.8px 2.2px rgba(0, 0, 0, 0.02),
        0 6.7px 5.3px rgba(0, 0, 0, 0.028), 0 12.5px 10px rgba(0, 0, 0, 0.035),
        0 22.3px 17.9px rgba(0, 0, 0, 0.042),
        0 41.8px 33.4px rgba(0, 0, 0, 0.05), 0 100px 80px rgba(0, 0, 0, 0.07);
}
.vue-grid-item.vue-draggable-dragging::before {
    opacity: 1;
    transition: opacity 500ms;
}
/* bg element // placeholder */
.vue-grid-item.vue-grid-placeholder {
    opacity: 1;
    background-color: #d3e0ec;
    border-radius: 5px;
    border: 3px solid #028aff;
    box-sizing: border-box;
}

.item__wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    /* justify-content: center; */
    width: 100%;
    height: 100%;

    font-weight: 600;
    color: #566d82;
}
.item__header {
    text-align: left;
    align-self: start;
    padding: 0 20px;
    width: calc(100% - 40px);
}
.item__content {
    display: flex;
    height: 100%;
    align-items: center;
}
.chartWrapper {
    height: 300px;
}
</style>
