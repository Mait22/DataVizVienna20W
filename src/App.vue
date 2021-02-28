<template>
  <div id="app">
    
      <div v-if="wrongSelection">
        <br>
        <p style="color:red"><b> {{ wrongSelectionMsg }} </b></p>
        <br>
      </div>	 

    <div class="container-fluid">

       <div class="row" v-if = "dataLoadingReady">
         <div class="col-md-1">
           <br>
          <button v-on:click="updateSelections">Confirm Selection</button>
        </div>
         
         <div class="col-md-2">
          <p>Select variable for <b>x-axis</b> <br>of the sctterplot</p>
          <multiselect v-model="xVariableSelection" :options="options" :multiple="false" track-by="name" label="name"></multiselect>
        </div>

        <div class="col-md-2">
          <p>Select variable for <b>y-axis</b> <br>of the sctterplot</p>
          <multiselect v-model="yVariableSelection" :options="options" :multiple="false" track-by="name" label="name"></multiselect>
        </div>

        <div class="col-md-3">
          <p>Select <b>first</b> variable <br>for line plot</p>
          
          <multiselect v-model="covidSelection1" :options="optionsCovid1" :multiple="false" track-by="name" label="name"></multiselect>
        </div>

        <div class="col-md-3">
          <p>Select <b>second</b> variable <br>for line plot</p>
         
          <multiselect v-model="covidSelection2" :options="optionsCovid2" :multiple="false" track-by="name" label="name"></multiselect>
        </div>
      </div> 

      <br>
      <div v-if = "!dataLoadingReady">
        <h1>Please wait! Fetching latest Covid-19 data. It might take minute or two …</h1>
        <h3>With great data comes great power but also slow computation... 
          As the dashboard makes available all the data from https://covid.ourworldindata.org/ 
          across all the variables and dates, 
          initial brushing in scatterplot is a bit slow.</h3> 
      </div>

      <div class="row" v-if = "dataLoadingReady">
        <div class="col-md-5">
          <ScatterplotNo1/>
        </div>
        <div class="col-md-7">
           <WorldMap/>
        </div>
      </div>
      <div class="row" v-if = "dataLoadingReady">
        <div class="col-lg-8" style="float:none;margin:auto;">
          <LineChart/>
        </div>
      </div>
      <br>



    </div>
  </div>
</template>

<script>
import ScatterplotNo1 from './components/ScatterplotNo1.vue';
import LineChart from './components/LineChart.vue';
//import YearSlider from './components/YearSlider.vue';
import WorldMap from './components/WorldMap.vue';
import axios from 'axios'
import Multiselect from 'vue-multiselect'

export default {
  name: 'App',
  components: {
    ScatterplotNo1, WorldMap, Multiselect, LineChart
  },
  data() {
    return {
      wrongSelection: false, 
      wrongSelectionMsg: null, 
      options: [
        { name: 'Population', varibale_technical_name: 'population' },
        { name: 'Population Density', varibale_technical_name: 'population_density' },
        { name: 'Median Age', varibale_technical_name: 'median_age' },
        { name: 'Aged over 65', varibale_technical_name: 'aged_65_older' },
        { name: 'Aged over 70', varibale_technical_name: 'aged_70_older' },
        { name: 'GDPpc', varibale_technical_name: 'gdp_per_capita' },
        { name: 'Cardiovascular death rate', varibale_technical_name: 'cardiovasc_death_rate' },
        { name: 'Diabets Prevelance', varibale_technical_name: 'diabetes_prevalence' },
        { name: 'Female Smokers', varibale_technical_name: 'female_smokers' },
        { name: 'Male Smokers', varibale_technical_name: 'male_smokers' },
        { name: 'Handwashing facalities', varibale_technical_name: 'handwashing_facilities' },
        { name: 'Life Expectancy', varibale_technical_name: 'life_expectancy' },
        { name: 'Human Development Index', varibale_technical_name: 'human_development_index' }
      ],

        optionsCovid1: [
        { name: 'Total cases', varibale_technical_name: 'total_cases', type: 'total' },
        { name: 'New cases', varibale_technical_name: 'new_cases', type: 'total' },
        { name: 'New cases smoothed', varibale_technical_name: 'new_cases_smoothed', type: 'total' },
        { name: 'Total deaths', varibale_technical_name: 'total_deaths', type: 'total' },
        { name: 'New deaths', varibale_technical_name: 'new_deaths',type: 'total' },
        { name: 'New deaths smoothed', varibale_technical_name: 'new_deaths_smoothed', type: 'total' },
        { name: 'Total cases per million', varibale_technical_name: 'total_cases_per_million', type: 'per' },
        { name: 'New cases per million', varibale_technical_name: 'new_cases_per_million', type: 'per' },
        { name: 'New cases per million smoothed', varibale_technical_name: 'new_cases_smoothed_per_million', type: 'per' },
        { name: 'Total deaths per million', varibale_technical_name: 'total_deaths_per_million', type: 'per' },
        { name: 'New deaths per million', varibale_technical_name: 'new_deaths_per_million', type: 'per' },
        { name: 'New deaths smoothed per million', varibale_technical_name: 'new_deaths_smoothed_per_million', type: 'per' },
        { name: 'Disease reproduction rate', varibale_technical_name: 'reproduction_rate', type: 'per' },
        { name: 'ICU patient', varibale_technical_name: 'icu_patients', type: 'total' },
        { name: 'ICU patients per million', varibale_technical_name: 'icu_patients_per_million', type: 'per' },
        { name: 'Hospital patients', varibale_technical_name: 'hosp_patients', type: 'total' },
        { name: 'Hospital patients per million', varibale_technical_name: 'hosp_patients_per_million', type: 'per' },
        { name: 'New tests', varibale_technical_name: 'new_tests' },
        { name: 'Total tests', varibale_technical_name: 'total_tests' },
        { name: 'Total tests per thousand', varibale_technical_name: 'total_tests_per_thousand' },
        { name: 'New tests per thousand', varibale_technical_name: 'new_tests_per_thousand' },
        { name: 'New tests - smoothed', varibale_technical_name: 'new_tests_smoothed' },
        { name: 'Stringency Index', varibale_technical_name: 'stringency_index' }
      ],

        optionsCovid2: [
        { name: 'Total cases', varibale_technical_name: 'total_cases' },
        { name: 'New cases', varibale_technical_name: 'new_cases' },
        { name: 'New cases smoothed', varibale_technical_name: 'new_cases_smoothed' },
        { name: 'Total deaths', varibale_technical_name: 'total_deaths' },
        { name: 'New deaths', varibale_technical_name: 'new_deaths' },
        { name: 'New deaths smoothed', varibale_technical_name: 'new_deaths_smoothed' },
        { name: 'Total cases per million', varibale_technical_name: 'total_cases_per_million' },
        { name: 'New cases per million', varibale_technical_name: 'new_cases_per_million' },
        { name: 'New cases per million smoothed', varibale_technical_name: 'new_cases_smoothed_per_million', type: 'per' },
        { name: 'Total deaths per million', varibale_technical_name: 'total_deaths_per_million' },
        { name: 'New deaths per million', varibale_technical_name: 'new_deaths_per_million' },
        { name: 'New deaths smoothed per million', varibale_technical_name: 'new_deaths_smoothed_per_million' },
        { name: 'Disease reproduction rate', varibale_technical_name: 'reproduction_rate' },
        { name: 'ICU patient', varibale_technical_name: 'icu_patients' },
        { name: 'ICU patients per million', varibale_technical_name: 'icu_patients_per_million' },
        { name: 'Hospital patients', varibale_technical_name: 'hosp_patients' },
        { name: 'Hospital patients per million', varibale_technical_name: 'hosp_patients_per_million' },
        { name: 'New tests', varibale_technical_name: 'new_tests' },
        { name: 'Total tests', varibale_technical_name: 'total_tests' },
        { name: 'Total tests per thousand', varibale_technical_name: 'total_tests_per_thousand' },
        { name: 'New tests per thousand', varibale_technical_name: 'new_tests_per_thousand' },
        { name: 'New tests - smoothed', varibale_technical_name: 'new_tests_smoothed' },
        { name: 'Stringency Index', varibale_technical_name: 'stringency_index' }
      ],


      xVariableSelection: null,
      yVariableSelection: null,
      covidSelection1: null, 
      covidSelection2: null
      }
  },
  methods: {
    async loadData() {
        try {
            this.$store.commit('changeRenderComponents', false);
            const result = await axios.get('https://covid.ourworldindata.org/data/owid-covid-data.json');

            this.$store.commit('setCovidData', result);
            this.$store.commit('changeRenderComponents', true);

        } catch (err) {
            console.log(err);
        }
    },

    updateSelections() {

      let allTrue = true
      if(this.covidSelection1 === null || this.covidSelection2 === null || this.xVariableSelection.name === null ||  this.yVariableSelection.name === null) {
        this.wrongSelection = true
        this.wrongSelectionMsg = 'One of the variables has been left unchecked'
        allTrue = false
      }
      else if(this.xVariableSelection.name == this.yVariableSelection.name) {
        this.wrongSelection = true
        this.wrongSelectionMsg = 'Scatterplot variables can\'t be the same'
        allTrue = false
      }
      else if(this.covidSelection1.name == this.covidSelection2.name) {
        this.wrongSelection = true
        this.wrongSelectionMsg = 'Covid-19 related variables can\'t be the same'
        allTrue = false
      }


      if(allTrue) {
        this.wrongSelection = false
        this.wrongSelectionMsg = null
        this.$store.commit('changexVariable', this.xVariableSelection.varibale_technical_name);
        this.$store.commit('changeyVariable', this.yVariableSelection.varibale_technical_name);
        this.$store.commit('changexVariableLabel', this.xVariableSelection.name);
        this.$store.commit('changeyVariableLabel', this.yVariableSelection.name);
        this.$store.commit('changeLineVariable1', this.covidSelection1);
        this.$store.commit('changeLineVariable2', this.covidSelection2)
      }


    }
    
  },
  computed: {
    dataLoadingReady: {
      get() {
        return this.$store.getters.getRenderComponents;
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

  },

  mounted() {
    this.loadData()
    //this.$store.dispatch('loadData');
  },
}
</script>
<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 20px;
}
</style>

