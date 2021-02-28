<template>

  <div class="vis-component" ref="chart">
    <div v-if="(countryLabel === null) === false">
      Country: {{countryLabel}}
    </div>
    <div v-if="(countryLabel === null) === true">
        <br>
    </div>
    <svg class="main-svg" :width="svgWidth" :height="svgHeight">
      <g class="bg-group" ref="bgGroup"></g>
      <g class="chart-group" ref="chartGroup">
        <g class="path-group" ref="pathsGroup"></g>
      </g>
    </svg>

    <div class="placeholder">
      <p><b>Click on the country to select or unselect. <br>Of note, only countries presented on the scatterplot are selectable.</b></p>
    </div>
  </div>
</template>


<script>

import worldMap from '@/assets/countries-110m.json';
import * as d3 from 'd3';
import * as topojson from 'topojson';


export default {
  name: 'ChoroplethMap',
  props: {
  },
  data() {
    return {
      countryLabel: null, 
      svgWidth: 1152,
      svgHeight: 600,
      svgPadding: {
        top: 20, right: 20, bottom: 20, left: 20,
      },
    }
  },
  mounted() {
    this.drawMap();
  },
  methods: {

    drawMap() {
      if (this.$refs.chart) this.svgWidth = this.$refs.chart.clientWidth;

      d3.select(this.$refs.chartGroup).attr(
        "transform",
        `translate(${this.svgPadding.left},${this.svgPadding.top})`
      )      

      const projection = d3.geoNaturalEarth1()
      .translate([(this.svgWidth - this.svgPadding.left) / 2, (this.svgHeight - this.svgPadding.top) / 2]) 
      .scale([190]); 

      const path = d3.geoPath()
      .projection(projection);

      const pathsGroup = d3.select(this.$refs.pathsGroup);

      //console.log(topojson.feature(worldMap, worldMap.objects.countries).features)

      const replacements = {
            "Central African Rep.": "Central African Republic",
            "Dem. Rep. Congo": "Democratic Republic of Congo",
            "United States of America": "United States",
            "Côte d'Ivoire": "Cote d'Ivoire", 
            "S. Sudan": "South Sudan", 
            "eSwatini": "Eswatini", 
            "Solomon Is.": "Solomon Islands", 
            "Macedonia": "North Macedonia",
            "Bosnia and Herz.": "Bosnia and Herzegovina", 
            "Dominican Rep.": "Dominica",
            "Eq. Guinea": "Guinea", 
            "Somaliland": "Somalia", 
            "Timor-Leste": "Timor"}

      let allCountries = this.scatterData.map(el => el.name)
      //console.log(allCountries)
      //console.log(this.scatterData)



      pathsGroup.exit().remove();


        pathsGroup.
        selectAll("path")
        .data( topojson.feature(worldMap, worldMap.objects.countries).features)        
        .enter()
        .append("path")
        .attr("d", path)
        .style("stroke", "#fff")
        .style("stroke-width", "0.75")
        .on('click',  (d) => {
          try{console.log(d.properties.name)
          let rep = d.properties.name
          if(d.properties.name in replacements) {
             rep = replacements[d.properties.name]
          }
          if(allCountries.includes(rep)) {
            if(this.selectedCountries.includes(rep)){
              let newArray = this.selectedCountries.filter(el => el !== rep)
              this.$store.commit("changeSelectedCountries", newArray)
            }
            else if(!this.selectedCountries.includes(rep)){
              let newArray = this.selectedCountries
              newArray.push(rep)
              this.$store.commit("changeSelectedCountries", newArray)
            }

          }}
          catch (err) {
            console.log('Error occured')
          }
              
        })

        .on('mouseover', d => {
          this.countryLabel = d.properties.name
          })
        .on('mouseout', () => {
          this.countryLabel = null
          })

        const combineCountries = ['']
        pathsGroup
        .selectAll("path")
        .style("fill", (d) => {
        if(this.selectedCountries.length !== 0){
          // Replacements
          if(d.properties.name in replacements){
            let rep = replacements[d.properties.name]
            return this.selectedCountries.includes(rep) ? 
            (this.scatterData.filter(el => el.name === rep)[0].color)
            : 
            '#808080' 
          }         

          // Matches
          if(!combineCountries.includes(d.properties.name) ){
            return this.selectedCountries.includes(d.properties.name) ? 
            (this.scatterData.filter(el => el.name === d.properties.name)[0].color)
            : 
            '#808080' 
          }
            
        }
        if(this.selectedCountries.length === 0){
          //console.log('Len 0')
          return '#808080'   
        }
       
      })
        
    }
  },
  computed: {
     
    selectedCountries: {
      get() {
        return this.$store.getters.selectedCountries;
      }
    },

    getAllCountries: {
      get() {
        return this.$store.getters.getAllCountries;
      }
    },


    scatterData: {
      get() {
        return this.$store.getters.scatterData;
      },
    },

  },
  watch: {
    selectedCountries: {
      handler() {
        this.drawMap();
      },
      deep: true,
    },
    
  
  },
}
</script>

<style>
</style>
