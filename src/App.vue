<template>
  <div class="container">
    <div class="currency-converter">
      <h2>VUE CONVERTER</h2>
      <div class="currency-converter__block">
        <p>Maximal USD is 10000</p>
        <div class="currency-converter__item">
          <input
            type="number"
w            min="0"
            max="10000"
            class="currency-converter__input"
            v-model="firstRateValue"
          />
          <select v-model="firstConverterSelected" name="select">
            <option selected="selected">
              {{ firstConverterCurrencies[0] }}
            </option>
          </select>
        </div>
        <div class="currency-converter__item">
          <input
            type="number"
            min="0"
            class="currency-converter__input"
            v-model="secondRateValue"
          />
          <select v-model="secondConverterSelected" name="select">
            <option v-for="item in secondConverterCurrencies" :key="item.index">
              {{ item }}
            </option>
          </select>
        </div>
      </div>
    </div>
    <div class="currency-exchange-block">
      <div>
        <div class="currency-exchange__buttons">
          <select v-model="firstExchangeSelected" name="select">
            <option v-for="item in secondConverterCurrencies" :key="item.index">
              {{ item }}
            </option>
          </select>
          <button
            class="currency-exchange-block__button"
            @click="fetchFunction"
          >
            Update CURRENCY
          </button>
          <button class="currency-exchange-block__button" @click="togglePopup">
            ADD CURRENCY
          </button>
        </div>
      </div>
      <div class="currency-exchange-list">
        <div>
          <h5 v-for="item in secondConverterCurrencies" :key="item.index">
            {{ item }}
          </h5>
        </div>
        <div>
          <h5 v-for="item in firtsRatesCurrencies" :key="item.index">
            {{ item }}
          </h5>
        </div>
      </div>

      <div class="currency-exchange-popup" v-if="currenciesPopup">
        <div class="currency-exchange-search">
          <input
            type="text"
            v-model="searchCurrencies"
            placeholder="try to search currencies"
          />
        </div>
        <button
          @click="addCurrency1"
          v-for="item in popupCurrencies"
          :key="item.index"
        >
          {{ item }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      firstRateValue: 0,
      secondRateValue: 0,

      firstConverterCurrencies: [],
      secondConverterCurrencies: [],

      firtsExchangeCurrencies: [],
      firtsRatesCurrencies: [],

      firstConverterSelected: "USD",
      secondConverterSelected: "EUR",
      firstExchangeSelected: "USD",

      currenciesPopup: false,
      searchCurrencies: "",
    };
  },
  methods: {
    togglePopup() {
      this.currenciesPopup = !this.currenciesPopup;
    },
    addCurrency1(event) {
      this.currenciesPopup = !this.currenciesPopup;
  
      if(!this.secondConverterCurrencies.includes(event.target.innerHTML)) {
        this.secondConverterCurrencies.push(event.target.innerHTML);
        this.firtsRatesCurrencies.push(this.currency[event.target.innerHTML].toFixed(2));
      } else {
        alert('Already added currency')
      }

      this.currenciesPopup = !this.currenciesPopup;

      localStorage.setItem("currencies", this.secondConverterCurrencies);
      localStorage.setItem("rates", this.firtsRatesCurrencies);
    },
    fetchFunction() {
      this.firtsExchangeCurrencies = [];

      axios
        .get(
          "https://api.freecurrencyapi.com/v1/latest?apikey=xAJJkg1XajGj4NYy7k7kr8VgVeZ4XaboF4g01Rr4"
        )
        .then((response) => {
          this.currency = response.data.data

          this.firstConverterCurrencies = Object.keys(this.currency).filter(
            (el) => el === "USD"
          );

          this.secondConverterCurrencies = Object.keys(this.currency).filter(
            (el) =>
              el === "USD" ||
              el === "EUR" ||
              el === "BGN" ||
              el === "GBP" ||
              el === "HKD"
          );

          if (localStorage.getItem("currencies") != null) {
            this.secondConverterCurrencies = localStorage.currencies.split(",");
          } else {
            for (var el in this.currency) {
              if (
                el === "USD" ||
                el === "EUR" ||
                el === "BGN" ||
                el === "GBP" ||
                el === "HKD"
              ) {
                this.firtsExchangeCurrencies.push(this.currency[el].toFixed(2));
                
              }
            }
          }

          this.actualRates5 = Object.keys(this.currency)
        });
    },
  },
  computed: {
    popupCurrencies() {
      return this.actualRates5.filter((post) => {
        return post.toUpperCase().includes(this.searchCurrencies.toUpperCase());
      });
    },
  },

  watch: {
    firstRateValue: function (val) {
      if (val > 10000) {
        this.firstRateValue = 10000;
      } else {
        this.secondRateValue =
          (val * this.currency[this.secondConverterSelected]) /
          this.currency[this.firstConverterSelected];
      }
    },

    secondRateValue: function (val) {
      this.firstRateValue =
        (val * this.currency[this.firstConverterSelected]) /
        this.currency[this.secondConverterSelected];
    },

    secondConverterSelected: function (val) {
      this.secondRateValue = this.firstRateValue * this.currency[val];
    },

    firstExchangeSelected: function (val) {
      this.firtsRatesCurrencies = [];

      if (localStorage.getItem("rates") != null) {
        const storageRates = localStorage.rates.split(",");
        localStorage.rates.split(",").forEach((el, id) => {
          this.firtsRatesCurrencies.push(
            (1 * (this.currency[val] / storageRates[id])).toFixed(2)
          );
        });
      } else {
        this.firtsExchangeCurrencies.forEach((el, id) => {
          this.firtsRatesCurrencies.push(
            (
              1 *
              (this.currency[val] / this.firtsExchangeCurrencies[id])
            ).toFixed(2)
          );
        });
      }
    },
  },
  created() {
    this.fetchFunction();

    if (localStorage.getItem("rates") != null) {
      this.firtsRatesCurrencies = localStorage.rates.split(",");
    } else {
      this.firtsRatesCurrencies = this.firtsExchangeCurrencies;
    }
  },
};
</script>

<style lang="scss">
body {
  text-align: center;
  padding: 0;
  margin: 0;
  height: auto;
  color: black;
}

h2 {
  padding-top: 20px;
  margin-top: 10px;
}
.container {
  margin: 0 auto;
  width: 320px;
  height: 100%;
  background-color: white;
  border: 1px solid black;
  box-shadow: 0 0 10px 5px rgba(221, 221, 221, 1);
}

input {
  display: block;
  width: 150px;
  height: 20px;
  padding: 5px 5px;
  font-family: inherit;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
  outline: none;
  color: black;
  background-color: #ffffff;
  background-clip: padding-box;
  border: 1px solid black;

  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

select {
  height: 32px;
  border: 1px solid black;
}

.currency-converter__item {
  padding-bottom: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.currency-exchange-block {
  display: flex;
  flex-direction: column;
}

.currency-exchange__buttons {
  padding-bottom: 20px;
  display: flex;
  flex-direction: column;
}

.currency-exchange-list {
  display: flex;
  margin: 0 auto;
  margin-bottom: 10px;
  border: 1px solid black;
  h5 {
    margin: 5px;
  }
}

.currency-exchange-popup {
  position: absolute;
  top: 10px;
  width: 320px;
  height: 237px;
  background-color: white;
  box-shadow: 0 0 10px 5px rgba(221, 221, 221, 1);
}
.currency-exchange-search {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
  padding: 15px;
  input {
    margin: 5px auto;
  }
}
</style>
