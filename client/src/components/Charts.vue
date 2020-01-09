<template>
  <div class="content">
    <div class="container"> 
        <div class="Search__container">       
            <input
                v-model="newPVValue"
                class="Search__input"
                @keyup.enter="postData"
                placeholder="Enter PV Power"
                type="search" name="search">
            <input
                v-model="newGridValue"
                class="Search__input"
                @keyup.enter="postData"
                placeholder="Enter Grid Power"
                type="search" name="search">
          <button class="Search__button" @click="postData">Add Data Point</button>          
        <div class="error-message" v-if="showError">
            {{ error }}
        </div>
      </div>     
      <hr>
      <div class="Chart__container">
        <div class="Chart__title">
          <h2>Power of PV and Grid </h2>
          <button class="Search__button" @click="getData"> Reload Chart </button>

        </div>
        <hr>
        <div class="Chart__content">
          <GradientLineChart chart-id="line-daily" v-if="loaded" :chartData="chartData" :refresh="refresh"/>          
        </div>
      </div>

      <div class="Chart__container">
        <div class="Chart__title">
          <h2> Household Power Usage </h2>
          <button class="Search__button" @click="getData"> Reload Chart </button>

        </div>
        <hr>
        <div class="Chart__content">
          <StackedLineChart chart-id="line-stacked1" v-if="loaded" :chartData="chartDataStacked" :refresh="refresh"/>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
  import StackedLineChart from './StackedLineChart'
  import GradientLineChart from './GradientLineChart'
  import PostService from '../PostService'

  export default {
    components: { 
        GradientLineChart,
        StackedLineChart
    },
    data () {
      return {
          chartData: '',
          chartStacked: '',
          refresh: false,
          error: '',
          newPVValue: null,
          newGridValue: null,
          showError: false,
          loaded: false                   
      }
    },

    async created() {
        this.getData()        
    },    
    methods: {
        async postData() {
            if (this.newPVValue == null || this.newGridValue == null ) {
                this.error = 'Please enter a valid value'
                this.showError = true
            } else {
                this.showError = false
                await PostService.insertPost(parseFloat(this.newPVValue),
                                              parseFloat(this.newGridValue) )
                this.getData()
            }
        },        
        async getData() {
            try {
                this.loaded = false
                 var rawData = await PostService.getPosts()
                 var timestamps = rawData.map(entry => entry.timestamp)
                 var pv = rawData.map(entry => entry.pv)
                 var grid = rawData.map(entry => entry.grid)
                 var pv_stacked = pv.map(function(entry, i) {    //stackedPV
                    return entry + grid[i];
                    });                  
                 var zippedPV = pv.map(function(entry, i) {
                    return {x: timestamps[i], y: entry};
                    });
                var zippedPV_stacked = pv_stacked.map(function(entry, i) {
                    return {x: timestamps[i], y: entry};
                    });
                var zippedGrid = grid.map(function(entry, i) {
                    return {x: timestamps[i], y: entry};
                    })
                 
                 this.chartData = {                     
                     datasets: [
                         {label: 'Grid',
                         yAxisID : 'y-axis-0',
                         data: zippedGrid
                         },
                         {label: 'PV',
                         yAxisID : 'y-axis-0',
                         data: zippedPV
                         }]
                 }
                 this.chartDataStacked = {                     
                     datasets: [
                         {label: 'Grid',
                         yAxisID : 'y-axis-0',
                         data: zippedGrid
                         },
                         {label: 'PV',
                         yAxisID : 'y-axis-0',
                         data: zippedPV_stacked
                         }]
                 }
                 setTimeout(() => {
                     this.loaded = true
                    this.refresh = !this.refresh
                }, 100)
            } catch(err) {
                this.error = err.message
            }
        }
    }
  }
</script>

<style lang="scss">
  @import "../assets/styles/_variables.scss";

  hr {
    display: block;
    height: 1px;
    border: 0;
    border-top: 1px solid color(solitude);
    margin: 0.3em 0;
    padding: 0;
  }

  .content {
    background: color(ghost-white);
    min-height: calc(100vh - 207px);
  }

  .title {
    text-align: center;
    color: color(fjord);
  }

  .container {
    @include container();
  }

  .loading {
    text-align: center;
    color: color(fjord);
    font-size: rem(18);
  }

  .error-message {
    text-align: center;
    color: #ff6633;
  }

  .Search {
    @include has(container) {
      max-width: rem(640);
      padding: rem(20) 0 rem(20) 0;
      margin: 0 auto;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;

      @include media($sm-up) {
        flex-direction: row;
      }
    }
    @include has(input) {
      padding: rem(10);      
      background-color: #fff;
      border: 1px solid color(solitude);
      font-size: rem(16);
      flex: 1;
    }

    @include has(button) {
      appearance: none;
      padding:rem(10) rem(3);
      margin: rem(5) rem(20);
      border: 1px #c5ae92;
      border-radius: 3px;
      cursor: pointer;
      text-align: center;
      font-size:15;
      font-weight: 300;
      color: #fff;
      background: #d8a770;

      @include hover-active-states {
        background: darken(#cfb495, 10%);
      }
    }

    @include has(icon) {
      width: rem(40);
      height: rem(40);
      margin-left: rem(15);
      cursor: pointer;
      color: color(fjord);

      @include hover-active-states {
        color: color(robin-egg-blue);
      }

      > svg {
        fill: currentColor;
      }
    }

    @include has(settings) {
      max-width: rem(640);
      margin: 0 auto;
      padding: rem(20) 0 rem(5) 0;
      display: flex;
      flex-direction: row;
      justify-content: center;

        .vdp-datepicker + .vdp-datepicker {
          margin-left: 1.25rem;
          flex: 1;
        }
    }
  }

  .Chart__container {
    border-radius: $base-border-radius;
    background-color: #fff;
    box-shadow: 0 15px 30px 0 rgba(0,0,0,.11), 0 5px 15px 0 rgba(0,0,0,.08);
    padding: rem(20) rem(40);
    margin: rem(30) 0;
  }

  .Chart__title {
    display: flex;
    flex-direction: row;
    // margin-bottom: rem(20);
    justify-content: space-between;

    h2 {
      display: flex;
      align-items: center;
      color: color(fjord);
      margin: 0;
      font-weight: 600;
      font-size: rem(16);

      > span {
        font-weight: 400;
        color: #5d5b6a;
        font-size: rem(16);
        margin-left: rem(25);
      }
    }
    
    .Search__input{
      padding: rem(5)
    }

    
  }

  .sk-cube-grid {
    width: 40px;
    height: 40px;
    margin: 100px auto;
  }

  .sk-cube-grid .sk-cube {
    width: 33%;
    height: 33%;
    background-color: color(orange);
    float: left;
    -webkit-animation: sk-cubeGridScaleDelay 1.3s infinite ease-in-out;
            animation: sk-cubeGridScaleDelay 1.3s infinite ease-in-out;
  }
  .sk-cube-grid .sk-cube1 {
    -webkit-animation-delay: 0.2s;
            animation-delay: 0.2s; }
  .sk-cube-grid .sk-cube2 {
    -webkit-animation-delay: 0.3s;
            animation-delay: 0.3s; }
  .sk-cube-grid .sk-cube3 {
    -webkit-animation-delay: 0.4s;
            animation-delay: 0.4s; }
  .sk-cube-grid .sk-cube4 {
    -webkit-animation-delay: 0.1s;
            animation-delay: 0.1s; }
  .sk-cube-grid .sk-cube5 {
    -webkit-animation-delay: 0.2s;
            animation-delay: 0.2s; }
  .sk-cube-grid .sk-cube6 {
    -webkit-animation-delay: 0.3s;
            animation-delay: 0.3s; }
  .sk-cube-grid .sk-cube7 {
    -webkit-animation-delay: 0s;
            animation-delay: 0s; }
  .sk-cube-grid .sk-cube8 {
    -webkit-animation-delay: 0.1s;
            animation-delay: 0.1s; }
  .sk-cube-grid .sk-cube9 {
    -webkit-animation-delay: 0.2s;
            animation-delay: 0.2s; }

  @-webkit-keyframes sk-cubeGridScaleDelay {
    0%, 70%, 100% {
      -webkit-transform: scale3D(1, 1, 1);
              transform: scale3D(1, 1, 1);
    } 35% {
      -webkit-transform: scale3D(0, 0, 1);
              transform: scale3D(0, 0, 1);
    }
  }

  @keyframes sk-cubeGridScaleDelay {
    0%, 70%, 100% {
      -webkit-transform: scale3D(1, 1, 1);
              transform: scale3D(1, 1, 1);
    } 35% {
      -webkit-transform: scale3D(0, 0, 1);
              transform: scale3D(0, 0, 1);
    }
  }
</style>
