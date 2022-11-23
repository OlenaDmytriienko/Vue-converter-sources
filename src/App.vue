<script setup>
import { RouterLink, RouterView } from 'vue-router'
import './style.scss'

</script>

<template>
<div id="app" class="modal-vue">
<header>
  <h1>Currency converter</h1>
  <div class="header__cur">
  <button @click="refresh()" :disabled="pausedFor5Seconds">Refresh</button>
</div>
</header>
<div class="page__wrapper">
<div class="main_container">
  <div class="select-container">
    <input type="text" id="name" name="name" class="form-control" @input="convertInput($event)" :value="inputValue">
    <select id="country1" @change="convertSelectInput($event)">
      <option value="USD">USD</option>
      <option value="UAH">UAH</option>
      <option value="EUR">Euro</option>
    </select>
    to
    <select id="country2" @change="convertSelectOutput($event)">
      <option value="USD">USD</option>
      <option value="UAH">UAH</option>
      <option value="EUR">Euro</option>
    </select>
    <div  class="alert">
    <div class="cur" v-show="showError"> Name must be less than 5 characters long.</div>
    </div>
  </div>
  <div class="pop">
      <div class="pop__cur">
      <h3>Currency</h3>
      <h3>Rate</h3>
     </div>
     <div class="pop__cur">
      <p>USD</p>
      <p>{{Math.round((apiResponseArr?.rates?.USD ?? 0) * 100)/100}}</p>
     </div>
      <div class="pop__cur">
      <p>EUR</p>
      <p>{{Math.round((apiResponseArr?.rates?.EUR?? 0) * 100)/100}}</p>
     </div>
      <div class="pop__cur">
      <p>UAH</p>
      <p>{{Math.round((apiResponseArr?.rates?.UAH?? 0) * 100)/100}}</p>
     </div>
      <div class="pop__cur">
      <p>BTC</p>
      <p>{{apiResponseArr?.rates?.BTC ?? 0}}</p>
     </div>
      <div class="pop__cur" v-show="savedArr.includes('CHF')">
      <p>CHF</p>
      <p>{{Math.round((apiResponseArr?.rates?.CHF?? 0) * 100)/100}}</p>
     </div>
         <div class="pop__cur" v-show="savedArr.includes('GBP')">
      <p>GBP</p>
      <p>{{Math.round((apiResponseArr?.rates?.GBP?? 0) * 100)/100}}</p>
     </div>
         <div class="pop__cur" v-show="savedArr.includes('TRY')">
      <p>TRY</p>
      <p>{{Math.round((apiResponseArr?.rates?.TRY?? 0) * 100)/100}}</p>
     </div>
         <div class="pop__cur" v-show="savedArr.includes('JPY')">
      <p>JPY</p>
      <p>{{Math.round((apiResponseArr?.rates?.JPY?? 0) * 100)/100}}</p>
     </div>
         <div class="pop__cur" v-show="savedArr.includes('AED')">
      <p>AED</p>
      <p>{{Math.round((apiResponseArr?.rates?.AED?? 0) * 100)/100}}</p>
     </div>
     </div>
       <button @click="showModal = true">Search currency</button>

    <div class="result">
      <p>{{inputValue}}</p>
      <p>=</p>
      <p>{{result}}</p>
    </div>
  </div>
</div>
  <div class="modal" v-if="showModal">
    <button class="close" @click="showModal = false">x</button>
    <p class="pop__p">Choose currencies</p>
      <div class="pop__curr" >
      <button v-for="curr in settingsCurrArr" @click="addCurrToLocalStorage(curr)"> {{curr}}</button>
      </div>
  </div>

</div>
</template>

<script>
export default {
  name: 'App',
  el: '#app',

 data () {
    return {
      api: 'https://api.exchangerate.host/latest?base=',
        headers: {'Content-Type': 'application/json'},
      // currencyjson: any = [],

  inputCurrency: 'USD',
  outputCurrency : 'USD',
  apiResponseArr: [],
  inputValue : '',
  outputValue: 1,
  uahToEuro: 1,
  uahToUsd: 1,
  result: '',
  settingsCurrArr: ['CHF', 'GBP','TRY', 'AED', 'JPY'],
  showModal: false,
  showError: false,
  savedArr: [],
  pausedFor5Seconds: false,
    }
  },
methods: {
      async requestTo(api) {
      try {
        const response = await fetch(api, {
          method: 'GET',
          headers: this.headers,
        }).then(this.checkStatus)
            .then(this.parseJSON);
          this.apiResponseArr = response;
          console.log(this.apiResponseArr.rates.USD);
          this.setupOutput()
      } catch (error) {
        this.error = error
      }
    },
     checkStatus: function (resp) {
      if (resp.status >= 200 && resp.status < 300) {
        return resp;
      }
      return this.parseJSON(resp).then((resp) => {
        throw resp;
      });
    },
      parseJSON: function (resp) {
        return (resp.json ? resp.json() : resp);
    },
  convertInput(event){
    const value = event.target.value
      console.log(value, this.inputValue)
      if (String(value).length <= 5) {
        this.inputValue = value
        this.requestTo(this.api + this.inputCurrency)
        this.showError = false
      }
      else {
        this.showError = true
      }
        this.$forceUpdate()
        console.log(this.inputValue);
   },
    convertSelectInput(event){
    this.inputCurrency = event.target.value
    console.log(this.inputCurrency);
    this.requestTo(this.api + this.inputCurrency)
   },
    convertSelectOutput(event){
    this.outputCurrency = event.target.value
    console.log(this.outputCurrency);
    this.requestTo(this.api + this.inputCurrency)
   },
async setupOutput(){
      if (this.outputCurrency == 'USD'){
        this.outputValue = Number(this.apiResponseArr.rates.USD)
      }
      else if (this.outputCurrency == 'UAH'){
        this.outputValue = Number(this.apiResponseArr.rates.UAH)
      }
      else if (this.outputCurrency == 'EUR'){
        this.outputValue = Number(this.apiResponseArr.rates.EUR)
      }
      this.result = this.outputValue*this.inputValue
      this.result = Math.round(this.outputValue * Number(this.inputValue)*100)/100
      console.log('result ' + this.result);
        console.log(this.apiResponseArr.rates);
    },
   addCurrToLocalStorage(curr){
   this.savedArr = localStorage.settingsCurrArr.split(',').map(str => { return str;})

console.log('temp' + this.savedArr);
const index = this.savedArr.indexOf(curr);
if (index > -1) { // only splice array when item is found
this.savedArr.splice(index, 1); // 2nd parameter means remove one item only
} else {
  this.savedArr.push(curr)
}
localStorage.settingsCurrArr = this.savedArr
console.log(localStorage.settingsCurrArr);
   }, 
async refresh(){
  this.pausedFor5Seconds = true;
  setTimeout(() => this.pausedFor5Seconds = false, 5000);
  this.requestTo(this.api + this.inputCurrency)
  console.log('refresh');
}
},
mounted() {
   this.savedArr = localStorage.settingsCurrArr.split(',').map(str => { return str;})
    },
  }

</script>
