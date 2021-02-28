<template>

  <div class="vis-component" ref="chart" >

      <b><p>Specify countries based on their socioeconomic profile.</p></b>
      <svg class="main-svg" :width="svgWidth" :height="svgHeight">
        <g class="chart-group" ref="chartGroup">
          <g class="axis axis-x" ref="axisX"></g>
          <g class="axis axis-y" ref="axisY"></g>
          <g class="scatter-group" ref="scatterGroup"></g>
        </g>
      </svg>

  </div>
</template>

<script>
import * as d3 from "d3";
//import axios from 'axios'


export default {
  name: "Scatterplot",
  props: {},
  data() {
    return {
      svgWidth: 750,
      svgHeight: 500,
      svgPadding: {
        top: 35,
        right: 100,
        bottom: 40,
        left: 55,
      },
    };
  },
  mounted() {
    this.drawChart()
  },

  methods: {

     drawChart() {

      if (this.$refs.chart) { this.svgWidth = this.$refs.chart.clientWidth }
      d3.select(this.$refs.chartGroup).attr(
        "transform",
        `translate(${this.svgPadding.left},${this.svgPadding.top})`
      );
      console.log('Drawing data')
      this.$store.commit("changeSelectedCountries", [])
      this.drawXAxis();
      this.drawYAxis();
      this.drawSquares();
      this.addBrush();
      this.drawCircles();


    },

    drawXAxis() {
      d3.select("#xAxisText").remove()

      d3.select(this.$refs.axisX)
        .attr(
          "transform",
          `translate( 0, ${
            this.svgHeight - this.svgPadding.top - this.svgPadding.bottom
          } )`
        )
        .call(d3.axisBottom(this.xScale))
        .append("text")
        .attr("id", "xAxisText")
        .attr("x", this.svgWidth - this.svgPadding.left)
        .attr("y", this.svgPadding.bottom*0.9)
        .attr("text-anchor", "end")
        .attr("fill", "black")
        .attr("font-size", "9")
        .text(this.getxVariableLabel);
    },

    drawYAxis() {
      d3.select("#yAxisText").remove()

      d3.select(this.$refs.axisY)
        .call(d3.axisLeft(this.yScale))
        .append("text")
        .attr("id", "yAxisText")
        .attr("transform", "rotate(-90)")
        .attr("y", -this.svgPadding.left)
        .attr("dy", "0.71em")
        .attr("text-anchor", "end")
        .attr("fill", "black")
        .attr("font-size", "9")
        .text(this.getyVariableLabel);
    },

    drawCircles() {
      const scatterGroup = d3.select(this.$refs.scatterGroup);


      const div = d3
      .select(this.$refs.chart)
      .append('div')
      .attr('class', 'tooltip')
      .style('opacity', 0)
      .style("background-color", "white");


      const circle = scatterGroup
        .selectAll("circle")
        .data(this.scatterData, (d) => {
          d.name;
        });

      circle.exit().remove();

      circle
        .enter()
        .append("circle")
        .attr("cx", (d) => this.xScale(d.valueX))
        .attr("cy", (d) => this.yScale(d.valueY))
        .attr("r", 3.0)
        .attr('fill','blue')
        .attr('fill', d => d.highlight === 1 ? 'blue' : d.color)
        .attr('stroke', 'black')
        .attr("stroke-width",2)
        .on('mouseover', d => {
      div
        .transition()
        .duration(100)
        .style('opacity', 0.9);
      div
        .html('  Country: ' + "<b>" + d.name + "</b>")
        .style('left', (this.xScale(d.valueX) + 35) + 'px')
        .style('top', (this.yScale(d.valueY) + 35) + 'px');
      })
      .on('mouseout', () => {
      div
        .transition()
        .duration(200)
        .style('opacity', 0);
      })


 
    },


    drawSquares() {

      const scatterGroup = d3.select(this.$refs.scatterGroup);

      const square = scatterGroup
        .selectAll('rect')
        .data(this.squareData, d => d.name);
      
      square.exit().remove();

      square.enter()
            .append('rect')
            .attr('x', d => this.xScale(d.xStart))
            .attr('width',  (this.svgWidth - this.svgPadding.left - this.svgPadding.right) / 3)
            .attr('y', d => this.yScale(d.yEnd))
            .attr('height', (this.svgHeight - this.svgPadding.top - this.svgPadding.bottom) / 3)
            .style('fill', d => d.color)
            .merge(square)

    },

    addBrush() {
      const brush = d3
        .brush()
        .extent([
          [0, 0],
          [
            this.svgWidth - this.svgPadding.right - this.svgPadding.left,
            this.svgHeight - this.svgPadding.bottom - this.svgPadding.top,
          ],
        ])
        .on("end", () => {
          const selection = d3.event.selection;

          if(selection === null) {
            this.$store.commit("changeSelectedCountries", []);
          }

          else {

          

          const x0 = this.xScale.invert(selection[0][0]);
          const x1 = this.xScale.invert(selection[1][0]);
          const y0 = this.yScale.invert(selection[1][1]);
          const y1 = this.yScale.invert(selection[0][1]);


          const filteredData = this.scatterData.filter(
            (el) =>
              el.valueX >= x0 &&
              el.valueX <= x1 &&
              el.valueY >= y0 &&
              el.valueY <= y1
          );

          const filteredCountries = [];
          filteredData.forEach((el) => filteredCountries.push(el.name));

          if(filteredCountries.length > 0){
            this.$store.commit("changeSelectedCountries", filteredCountries);
          }
          if(filteredCountries.length === 0){
            this.$store.commit("changeSelectedCountries", []);}

          }

          //console.log('LineVariableCall')
          //console.log(this.getLineVariableNo1)
        })



      const scatterGroup = d3.select(this.$refs.scatterGroup);
      scatterGroup
        //.selectAll("brush")
        //.remove()
        .append("g")
        .attr("class", "brush")
        .call(brush);
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
        return this.$store.getters.scatterData;
      },
    },

    squareData: {
      get() {
        return this.$store.getters.squareData;
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

    getLineVariableNo1: {
      get() {
        return this.$store.getters.lineVariableNo1;
      },
    },


    // Finding min and max values
    dataMaxX() {
      return d3.max(this.xAxisData, (d) => d.value);
    },
    dataMinX() {
      return d3.min(this.xAxisData, (d) => d.value);
    },

    dataMaxY() {
      return d3.max(this.yAxisData, (d) => d.value);
    },

    dataMinY() {
      return d3.min(this.yAxisData, (d) => d.value);
    },


    // Scales
    // y-scale
    yScale() {
      return d3
        .scaleLinear()
        .rangeRound([
          this.svgHeight - this.svgPadding.top - this.svgPadding.bottom,
          0,
        ])
        .domain([this.dataMinY < 0 ? 0 : this.dataMinY, this.dataMaxY]);
    },

    // x-scale
    xScale() {
      return d3
        .scaleLinear()
        .rangeRound([
          this.svgWidth - this.svgPadding.right - this.svgPadding.left,
          0,
        ])
        .domain([this.dataMaxX, this.dataMinX < 0 ? 0 : this.dataMinX]);
    },
  },

  watch: {
    
    scatterData: {
      handler() {
        this.drawChart();
      },
      deep: true,
    },
  },
};
</script>

<style></style>



