<template>
    <section class="section">
        <div class="container">
            <div class="columns">
                <div class="column is-4 is-offset-4">
                    <b-field>
                        <b-input
                            v-model="amount"
                            type="number"
                            min="1"
                            placeHolder="Amount"
                        ></b-input>
                    </b-field>
                    <b-field>
                        <b-select
                            @select="currencySelected"
                            placeholder="Select a currency"
                            v-model="base"
                            expanded
                        >
                            <option
                                v-for="currency in currencies"
                                :value="currency"
                                :key="currency"
                            >
                                {{ currency }}
                            </option>
                        </b-select>
                    </b-field>
                </div>
            </div>
            <div class="columns">
                <div class="column is-offset-4 is-4">
                    <b-table
                        :data="ratesList.length == 0 ? [] : ratesList"
                        :striped="true"
                        :hoverable="true"
                        :mobile-cards="true"
                    >
                        <template slot-scope="props">
                            <b-table-column field="currency" label="CURRENCY">
                                {{ props.row.currency }}
                            </b-table-column>
                            <b-table-column
                                field="rate"
                                label="RATE"
                                numeric=""
                            >
                                {{ props.row.rate }}
                            </b-table-column>
                        </template>
                    </b-table>
                </div>
            </div>
        </div>
    </section>
</template>

<script>
// @ is an alias to /src
import axios from "axios";

export default {
    name: "Home",
    data: function() {
        return {
            rates: [],
            ratesList: [],
            base: "EUR",
            oldBase: "EUR",
            amount: 1,
            currencies: [],
            originalRates: []
        };
    },
    watch: {
        base: function(value) {
            this.currencySelected(value);
        },
        amount: function(value) {
            this.amountEntered(value);
        }
    },
    methods: {
        amountEntered: function(value) {
            let arr = [];
            if (value != "") {
                for (let i = 0; i < this.originalRates.length; i++) {
                    let obj = {};
                    obj.rate = this.originalRates[i].rate * value;
                    obj.rate = obj.rate.toFixed(2);
                    obj.currency = this.originalRates[i].currency;
                    arr.push(obj);
                }
            } else {
                this.ratesList = this.originalRates;
            }
            this.ratesList = arr;
        },
        currencySelected: function(value) {
            this.base = value;
            this.amount = 1;
            if (this.base != this.oldBase) {
                this.oldBase = this.base;
                axios
                    .get(
                        "https://api.exchangeratesapi.io/latest?base=" +
                            this.base
                    )
                    .then(response => {
                        this.currencies = [];
                        this.ratesList = [];
                        this.currencies.push(value);
                        this.base = response.data.base;
                        this.rates = response.data.rates;
                        this.originalRates = [];
                        for (let currency in this.rates) {
                            if (currency != this.base) {
                                let obj = {};
                                obj.currency = currency;
                                this.currencies.push(currency);
                                obj.rate = this.rates[currency];
                                this.ratesList.push(obj);
                                this.originalRates.push(obj);
                            }
                        }
                    })
                    .catch(e => {
                        console.log(e);
                    });
            }
        }
    },
    created() {
        axios
            .get("https://api.exchangeratesapi.io/latest")
            .then(response => {
                this.base = response.data.base;
                this.rates = response.data.rates;
                this.currencies.push(this.base);
                for (let currency in this.rates) {
                    let obj = {};
                    obj.currency = currency;
                    this.currencies.push(currency);
                    obj.rate = this.rates[currency];
                    this.ratesList.push(obj);
                    this.originalRates.push(obj);
                }
            })
            .catch(e => {
                console.log(e);
            });
    }
};
</script>
