<template>

  <div class="vis-component" ref="chart" >

    <div class="placeholder" v-if = "!dataLoadingReady">
      <br/>
      <b>Please wait! Fetching latest Covid-19 data ... It can take a minute!</b>
    </div>
      <b><p>Covid-19 dynamics in selected countries</p></b>
      <svg class="main-svg" :width="svgWidth" :height="svgHeight">
        <g class="chart-group" ref="chartGroup1">
          <g class="axis axis-x" ref="axisX1"></g>
          <g class="axis axis-y" ref="axisY1"></g>
          <g class="scatter-group" ref="lineGroup1"></g>
        </g>
      </svg>

      <p v-if="hooverCountry !== null">
        Currently highlighting country <b>{{hooverCountry}}</b>
      </p>
      <p v-if="hooverCountry === null">
        Move mouse over line to highligh the country on charts <b>above</b> and <b>below</b>. 
      </p>

      <svg class="main-svg" :width="svgWidth" :height="svgHeight">
        <g class="chart-group" ref="chartGroup2">
          <g class="axis axis-x" ref="axisX2"></g>
          <g class="axis axis-y" ref="axisY2"></g>
          <g class="scatter-group" ref="lineGroup2"></g>
        </g>
      </svg>

      <p>
        Time series values less than 0 have been deleted during data fetching, as they are clearly data entry mistakes or statistical correction entries.  
      </p>

    <div class="placeholder" v-if = "dataLoadingReady">

    </div>

  </div>
</template>

<script>
import * as d3 from "d3";
//import axios from 'axios'


export default {
  name: "LineChart",
  props: {},
  data() {
    return {
      svgWidth: 1200,
      svgHeight: 300,
      svgPadding: {
        top: 40,
        right: 100,
        bottom: 80,
        left: 75,
      },
      hooverCountry: null,
    };
  },
  mounted() {
    this.drawChart()
  },

  methods: {

     drawChart() {
      
      if (this.$refs.chart) { this.svgWidth = this.$refs.chart.clientWidth }
        d3.select(this.$refs.chartGroup1).attr(
          "transform",
          `translate(${this.svgPadding.left},${this.svgPadding.top})`
        );

      if (this.$refs.chart) { this.svgWidth = this.$refs.chart.clientWidth }
        d3.select(this.$refs.chartGroup2).attr(
          "transform",
          `translate(${this.svgPadding.left},${this.svgPadding.top})`
        );

      //console.log('Drawing lines')
      //console.log(this.yScale2(1))


      this.drawXAxis1();
      this.drawYAxis1();

      this.drawXAxis2();
      this.drawYAxis2();

      this.drawLines1()
      this.drawLines2()


    

    },

    drawXAxis1() {
      d3.select("#xAxisText2").remove()

      d3.select(this.$refs.axisX1)
        .attr(
          "transform",
          `translate( 0, ${
            this.svgHeight - this.svgPadding.top - this.svgPadding.bottom
          } )`
        )
        .call(d3.axisBottom(this.xScale))
        .append("text")
        .attr("id", "xAxisText2")
        .attr("x", this.svgWidth*0.9)
        .attr("y", this.svgPadding.bottom*0.9)
        .attr("text-anchor", "end")
        .attr("fill", "black")
        .attr("font-size", "9")
        //.text('Date');
    },

    drawXAxis2() {
      d3.select("#xAxisText00").remove()

      d3.select(this.$refs.axisX2)
        .attr(
          "transform",
          `translate( 0, ${
            this.svgHeight - this.svgPadding.top - this.svgPadding.bottom
          } )`
        )
        .call(d3.axisBottom(this.xScale))
        .append("text")
        .attr("id", "xAxisText00")
        .attr("x", this.svgWidth*0.9)
        .attr("y", this.svgPadding.bottom*0.9)
        .attr("text-anchor", "end")
        .attr("fill", "black")
        .attr("font-size", "11")
        //.text('Date');
    },

    drawYAxis1() {
      d3.select("#yAxisText11").remove()

      d3.select(this.$refs.axisY1)
        .call(d3.axisLeft(this.yScale1))
        .append("text")
        .attr("id", "yAxisText11")
        .attr("transform", "rotate(-90)")
        .attr("y", -this.svgPadding.left)
        .attr("dy", "0.71em")
        .attr("text-anchor", "end")
        .attr("fill", "black")
        .attr("font-size", "11")
        .text(this.lineVariable1.name);
    },

    drawYAxis2() {
      d3.select("#yAxisText22").remove()

      d3.select(this.$refs.axisY2)
        .call(d3.axisLeft(this.yScale2))
        .append("text")
        .attr("id", "yAxisText22")
        .attr("transform", "rotate(-90)")
        .attr("y", -this.svgPadding.left)
        .attr("dy", "0.71em")
        .attr("text-anchor", "end")
        .attr("fill", "black")
        .attr("font-size", "11")
        .text(this.lineVariable2.name);
    },

    drawLines1() {
      console.log('Drawing lines')
      const lineGroup1 = d3.select(this.$refs.lineGroup1);


      const mouseoverHandler = (id, data) => {
        const selection = d3.selectAll(id)

        console.log(data)
        console.log(id)
      
        selection
        .attr('stroke','#4e99e4')
        .attr("stroke-width", 3)

        this.hooverCountry = data.countryName

        }


      const mouseoutHandler = (id, d) => {
        console.log(d)

        const selection = d3.selectAll(id)

        selection
          .attr('stroke','grey')
          .attr("stroke-width", 1.5)

        this.hooverCountry = null
        }

      const lineFunc = d3.line()
        //.curve(d3.curveNatural)
        .x(d => this.xScale(d.date))
        .y(d => this.yScale1(d.measure))

      const line = lineGroup1
        .selectAll('path')
        .data(this.lineData.data1, d => d.countryName)
      
      line.exit().remove();

      line.enter()
        .append('path')
        .attr("d", d => lineFunc(d.values))
        .attr("fill", "none")
        .attr("stroke", "grey")
        .attr("stroke-width", 1.5)
        .attr("id", (d) => `${d.countryName.replace(/[^A-Za-z0-9]/g, '')}`)
        .on("mouseover", (d) => mouseoverHandler(`#${d.countryName.replace(/[^A-Za-z0-9]/g, '')}`, d))
        .on("mouseout", (d) => mouseoutHandler(`#${d.countryName.replace(/[^A-Za-z0-9]/g, '')}`, d))
        .attr("stroke-linejoin", "round")
        .attr("stroke-linecap", "round")

 
    },

      drawLines2() {
      console.log('Drawing lines')
      const lineGroup1 = d3.select(this.$refs.lineGroup2);


      const mouseoverHandler2 = (id, data) => {
        const selection = d3.selectAll(id)

        console.log(data)
        console.log(id)
      
        selection
        .attr('stroke','#4e99e4')
        .attr("stroke-width", 3)

        this.hooverCountry = data.countryName

        }


      const mouseoutHandler2 = (id, d) => {
        console.log(d)

        const selection = d3.selectAll(id)

        selection
          .attr('stroke','grey')
          .attr("stroke-width", 1.5)

        this.hooverCountry = null
        }

      const lineFunc2 = d3.line()
        //.curve(d3.curveNatural)
        .x(d => this.xScale(d.date))
        .y(d => this.yScale2(d.measure))

      const line = lineGroup1
        .selectAll('path')
        .data(this.lineData.data2, d => d.countryName)
      
      line.exit().remove();

      line.enter()
        .append('path')
        .attr("d", d => lineFunc2(d.values))
        .attr("fill", "none")
        .attr("stroke", "grey")
        .attr("stroke-width", 1.5)
        .attr("id", (d) => `${d.countryName.replace(/[^A-Za-z0-9]/g, '')}`)
        .on("mouseover", (d) => mouseoverHandler2(`#${d.countryName.replace(/[^A-Za-z0-9]/g, '')}`, d))
        .on("mouseout", (d) => mouseoutHandler2(`#${d.countryName.replace(/[^A-Za-z0-9]/g, '')}`, d))
        .attr("stroke-linejoin", "round")
        .attr("stroke-linecap", "round")

 
    },
    
  },

  computed: {
    // Getting data
    xAxisData: {
      get() {
        return this.$store.getters.xAxisData;
      },
    },

    dataLoadingReady: {
      get() {
        return this.$store.getters.getRenderComponents;
      },
    },

    yAxisData: {
      get() {
        return this.$store.getters.yAxisData;
      },
    },

    scatterData: {
      get() {
        return this.$store.getters.scatterData2;
      },
    },

    squareData: {
      get() {
        return this.$store.getters.squareData2;
      },
    },

    selectedCountries: {
      get() {
        return this.$store.getters.selectedCountries;
      },
    },

    getxVariableLabel: {
      get() {
        return this.$store.getters.getxVariableLabel;
      },
    },

    getyVariableLabel: {
      get() {
        return this.$store.getters.getyVariableLabel;
      },
    },

    yVariable: {
      get() {
        return this.$store.getters.getxVariable;
      },
    },

    xVariable: {
      get() {
        return this.$store.getters.getyVariable;
      },
    },

    lineData: {
      get() {
        return this.$store.getters.lineVariableNo1;
      },
    },

    lineVariable1: {
      get() {
        return this.$store.getters.getLineVariable1;
      },
    },

    lineVariable2: {
      get() {
        return this.$store.getters.getLineVariable2;
      },
    },

    // Finding min and max values
    dataMaxX() {
      return d3.max(this.lineData.dateUniqueVal);
    },
    dataMinX() {
      return d3.min(this.lineData.dateUniqueVal);
    },

    dataMaxY1() {
      return d3.max(this.lineData.measureUniqueVal1);
    },

    dataMinY1() {
      return d3.min(this.lineData.measureUniqueVal1);
    },

    dataMaxY2() {
      return d3.max(this.lineData.measureUniqueVal2);
    },

    dataMinY2() {
      return d3.min(this.lineData.measureUniqueVal2);
    },

    

    // Scales
    // y-scale
    yScale1() {
      return d3
        .scaleLinear()
        .range([
          this.svgHeight - this.svgPadding.top - this.svgPadding.bottom,
          0,
        ])
        .domain([this.dataMinY1, this.dataMaxY1]);
        //.domain([this.dataMinY1 < 0 ? 0 : this.dataMinY1, this.dataMaxY1]);
    },

    yScale2() {
      return d3
        .scaleLinear()
        .range([
          this.svgHeight - this.svgPadding.top - this.svgPadding.bottom,
          0,
        ])
        .domain([this.dataMinY2, this.dataMaxY2]);
        //.domain([this.dataMinY2 < 0 ? 0 : this.dataMinY2, this.dataMaxY2]);
    },

    // x-scale
    xScale() {
      return d3
        .scaleTime()
        .rangeRound([
          this.svgWidth - this.svgPadding.right - this.svgPadding.left,
          0,
        ])
        //.domain(d3.extent(this.lineData.dateUniqueVal));
        .domain([this.dataMaxX, this.dataMinX]);

    },
  },

  watch: {
    
    lineData: {
      handler() {
        this.drawChart();
      },
      deep: true,
    },

    lineVariable1: {
      handler() {
        this.drawChart();
      },
      deep: true,
    },

    lineVariable2: {
      handler() {
        this.drawChart();
      },
      deep: true,
    },

    selectedCountries: {
      handler() {
        this.drawChart();
      },
      deep: true,
    },
  },
};
</script>

<style></style>



