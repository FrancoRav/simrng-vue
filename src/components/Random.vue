<template>
    <div class="form">
        <label>Distribución de Datos:</label>
        <select v-model="distribution">
            <option value="normal">Normal</option>
            <option value="uniform">Uniforme</option>
            <option value="exponential">Exponencial Negativa</option>
            <option value="poisson">Poisson</option>
        </select>
        <br>
        <label>Cantidad:</label>
        <input type="number" v-model="numGenerated">
        <span v-if="distribution == 'uniform'">
            <br>
            <label>Límite Inferior:</label>
            <input type="number" step="0.01" v-model="lowerLimit">
            <br>
            <label>Límite Superior:</label>
            <input type="number" step="0.01" v-model="upperLimit">
        </span>
        <span v-if="distribution == 'normal'">
            <br>
            <label>Media:</label>
            <input type="number" step="0.01" v-model="mean">
            <br>
            <label>Desviación Estándar:</label>
            <input type="number" step="0.01" v-model="sd">
            <br>
            <label>Algoritmo:</label>
            <select v-model="algorithm">
                <option value="boxmuller">Box-Müller</option>
                <option value="convolution">Convolution</option>
            </select>
        </span>
        <span v-if="distribution == 'poisson'">
            <br>
            <label>Lambda <span>λ</span>:</label>
            <input type="number" step="0.01" v-model="lambda">
        </span>
        <span v-if="distribution == 'exponential'">
            <br>
            <label>Lambda <span>λ</span>:</label>
            <input type="number" step="0.01" v-model="lambda">
            <br>
            <label>Media:</label>
            <input type="number" step="0.01" v-model="expmean">
        </span>
        <br>
        <button class="gen" @click="generateRandomNumbers" :disabled='inprogress'>Generar Números Aleatorios</button>
        <br>
        <label v-if="distribution != 'poisson'">Intervalos:</label>
        <input v-if="distribution != 'poisson'" type="number" v-model="intervals">
        <label>Significancia:</label>
        <div class="histgen">
            <br>
            <select v-model="significance">
                <option value=10>0.001</option>
                <option value=9>0.01</option>
                <option value=8>0.025</option>
                <option value=7>0.05</option>
                <option value=6>0.1</option>
                <option value=5>0.9</option>
                <option value=4>0.95</option>
                <option value=3>0.975</option>
                <option value=2>0.99</option>
                <option value=1>0.999</option>
            </select>
            <button class="gen" @click="regenIntervals" :disabled='inprogress'>↻</button>
        </div>
        <br>
        <div class="results" v-if="generated">
            <Pager class="page" v-if="generated" :totalPages="Math.ceil(generatedNumbers/30)" />
            <br>
            <div class="canvas">
                <canvas id="histogram"></canvas>
            </div>
            <div class="chi">
                <table>
                    <thead>
                        <tr>
                            <th>N°</th>
                            <th>Desde</th>
                            <th>Hasta</th>
                            <th>fo</th>
                            <th>fe</th>
                            <th>c</th>
                            <th>c(AC)</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(interval, index) in chi_table" :key="index">
                            <td>{{index+1}}</td>
                            <td>{{(index == 0 && last_dist == "normal") ? "-∞" : formatNumber(interval.lower)}}</td>
                            <td>{{(index == chi_table.length-1 && (last_dist == "normal" || last_dist == "exponential" || last_dist == "poisson")) ? "∞" : formatNumber(interval.upper)}}</td>
                            <td>{{interval.fo}}</td>
                            <td>{{formatNumber(interval.fe)}}</td>
                            <td>{{formatNumber(interval.c)}}</td>
                            <td>{{formatNumber(interval.c_ac)}}</td>
                        </tr>
                    </tbody>
                </table>
                <p>El chi-cuadrado calculado fue de {{formatNumber(Number(chi_calculated))}}, mientras que el valor crítico es de
                {{formatNumber(Number(chi_accepted))}}.</p>
                <p v-if="Number(chi_accepted) < Number(chi_calculated)">Por lo tanto, se rechaza la hipótesis de que la distribución es una {{ distribution_string }}</p>
                <p v-if="Number(chi_accepted) > Number(chi_calculated)">Por lo tanto, no se puede rechazar la hipótesis de que la distribución es una {{ distribution_string }}</p>
            </div>
        </div>
    </div>
</template>

<style>
.canvas {
    width: 100%;
}

.results {
    width: 100%;
}

.histgen {
    vertical-align: middle;
    display: flex;
    width: 100%;
}

canvas {
    width: 100%;
}

.form {
    display: flex;
    overflow-x: hidden;
    overflow-y: hidden;
    width: auto;
    flex-direction: column;
    align-items: flex-start;
    margin: 20px;
    padding: 20px;
    border: 1px solid #38444D;
    border-radius: 12px;
    background-color: #15202B;
    font-size: 16px;
    color: #E1E8ED;
}

.pagenr {
    float: left;
    display: block;
    text-align: center;
    padding: 4px 16px;
    text-decoration: none;
}

div.page {
    width: 100%;
}
div.numlist {
    width: 100%;
}

label {
    margin-bottom: 5px;
    color: #1DA1F2;
    font-weight: bold;
    text-transform: uppercase;
}

label span {
    text-transform: lowercase;
}

div.form>span {
    width: 100%;
}


input,
select {
    margin-bottom: 10px;
    padding: 10px;
    border: 1px solid #38444D;
    border-radius: 4px;
    font-size: 16px;
    width: 100%;
    color: #E1E8ED;
    background-color: #253341;
}

button {
    margin-bottom: 10px;
    padding: 10px;
    background-color: #1DA1F2;
    color: #fff;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    cursor: pointer;
}

button:hover {
    background-color: #0F8BAC;
}

button:disabled:hover {
    background-color: #333333;
}

button:disabled {
    background-color: #555555;
    color: #CCCCCC;
    cursor: default;
}


ul {
    margin: 0;
    padding: 0;
    list-style: none;
    /* add a fixed max-height for the panel */
    overflow-y: auto;
    /* add scrollbars when the content exceeds the panel's height */
    background-color: #15202B;
    /* add a white background color */
    border-radius: 4px;
    padding: 10px;
    box-shadow: 0 2px 4px rgba(255, 255, 255, 0.05), 0 0 1px rgba(255, 255, 255, 0.1);
    /* add a subtle box-shadow for depth */
    width: 100%;
}

li {
    margin-bottom: 5px;
    color: #fff;
    /* adjust the text color to be more visible */
    font-size: 18px;
}

.scroller {
    margin: 0;
    padding: 0;
    min-height: 200px;
    height: 200px;
    max-height: 200px;
    overflow-y: auto;
    background-color: #15202B;
    border-radius: 4px;
    padding: 10px;
    box-shadow: 0 2px 4px rgba(255, 255, 255, 0.05), 0 0 1px rgba(255, 255, 255, 0.1);
    width: 100%;
}

.resize-observer {
    position: absolute;
    top: -9999px;
    left: -9999px;
}

.num {
    margin: 0;
    display: flex;
}

h2 {
    margin: 20px 0 10px 0;
    font-size: 24px;
    color: #E1E8ED;
}

.form>span {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    margin-top: 10px;
}

.form>span>label {
    margin-bottom: 5px;
    color: #1DA1F2;
    font-weight: bold;
}

@media screen and (max-width: 768px) {
    .form {
        margin: 10px;
        padding: 10px;
    }

    input,
    select {
        width: 100%;
    }
}

/* Set the width and color of the scrollbar track */
::-webkit-scrollbar {
    width: 10px;
    background-color: #333333;
}

/* Set the appearance of the scrollbar thumb */
::-webkit-scrollbar-thumb {
    background-color: #666666;
    border-radius: 10px;
}

/* Set the appearance of the scrollbar thumb when hovering */
::-webkit-scrollbar-thumb:hover {
    background-color: #999999;
}

/* Set the width and color of the scrollbar track */
::-moz-scrollbar {
    width: 10px;
    background-color: #333333;
}

/* Set the appearance of the scrollbar thumb */
::-moz-scrollbar-thumb {
    background-color: #666666;
    border-radius: 10px;
}

/* Set the appearance of the scrollbar thumb when hovering */
::-moz-scrollbar-thumb:hover {
    background-color: #999999;
}

.table-container {
    width: 100%;
}

table {
    border-collapse: collapse;
    width: 100%;
    margin-top: 20px;
    margin-bottom: 20px;
}

th,
td {
    padding: 12px;
    text-align: left;
    border-bottom: 1px solid #38444D;
    color: #E1E8ED;
}

th {
    background-color: #15202B;
    text-transform: uppercase;
    font-size: 16px;
}

tr:nth-child(even) {
    background-color: #253341;
}

tr:nth-child(odd) {
    background-color: #15202B;
}

tr:hover {
    background-color: #1DA1F2;
    color: #fff;
}

.table-scroll {
    max-height: 400px;
    overflow-y: auto;
}
</style>

<script>
import Chart from 'chart.js/auto';
import 'chart.js';
import Pager from './Pager.vue';
import useValidate from '@vuelidate/core';
import { required, minValue, requiredIf } from '@vuelidate/validators';
export default {
    data() {
        return {
            v$: useValidate(),
            isinprogress: false,
            generated: false,
            distribution: 'normal',
            intervals: 5,
            significance: 7,
            seed: 0,
            numGenerated: 10,
            lowerLimit: 0,
            upperLimit: 1,
            chi_calculated: null,
            chi_accepted: null,
            chi_table: null,
            mean: 10,
            expmean: "",
            last_mean: null,
            last_sd: null,
            last_lower: null,
            last_upper: null,
            last_lambda: null,
            last_dist: null,
            sd: 1,
            algorithm: 'boxmuller',
            lambda: 5,
            generatedNumbers: 0,
            chart: null,
        };
    },
    mounted() {
    },
    components: {
        Pager
    },
    computed: {
        inprogress() {
            return this.isinprogress == true;
        },
        distribution_string() {
            switch (this.last_dist) {
                case 'normal':
                    return "Normal con media " + this.last_mean + " y desviación estándar " + this.last_sd ;
                case 'exponential':
                    return "Exponencial con lambda " + this.last_lambda;
                case 'poisson':
                    return "Poisson con lambda " + this.last_lambda;
                case 'uniform':
                    return "Uniforme entre " + this.last_lower + " y " + this.last_upper ;
            }
        }
    },
    validations() {
        return {
            distribution: { required },
            numGenerated: { required, minValue: minValue(1) },
            intervals: { required, minValue: minValue(1) },
            significance: { required },
            mean: { required: requiredIf(this.distribution == "normal") },
            sd: { required: requiredIf(this.distribution == "normal"), minValue: minValue(this.distribution == "normal" ? 0 : -Infinity) },
            lambda: { requiredPoi: requiredIf(this.distribution == "poisson"),
                    requiredExp: requiredIf(this.distribution == "exponential" && this.expmean == "")},
            expmean: { required: requiredIf(this.distribution == "exponential" && this.lambda == "")},
            lowerLimit: { required: requiredIf(this.distribution == "uniform")},
            upperLimit: { required: requiredIf(this.distribution == "uniform"), minValue: minValue(this.distribution == "uniform" ? this.lowerLimit : -Infinity)},
        }
    },
    methods: {
        disableButtons() {
            this.isinprogress = true;
        },
        enableButtons() {
            this.isinprogress = false;
        },
        regenIntervals() {
            this.getStatistics();
        },
        async getStatistics() {
            this.disableButtons();
            if (this.chart) {
                this.chart.destroy();
            }

            var reqdata = {
                intervals: this.intervals,
                significance: Number(this.significance),
            }
            console.log(reqdata);
            var myHeaders = new Headers();
            myHeaders.append("Content-Type", "application/json");
            var url = "http://127.0.0.1:3000/api/statistics";
            var requestOptions = {
                method: 'POST',
                headers: myHeaders,
                redirect: 'follow',
            };
            requestOptions.body = JSON.stringify(reqdata);
            var interval_size = 0;
            var interval_list = [];
            var data_list = [];
            var min_value = 0;
            var max_value = 0;
            var chi_calculated = 0;
            var chi_accepted = 0;
            await fetch(url, requestOptions)
                .then(response => response.json())
                .then(result => {
                    var histogram = result.histogram;
                    interval_list = histogram.x;
                    data_list = histogram.y;
                    interval_size = histogram.size;
                    min_value = histogram.lower;
                    max_value = histogram.upper;
                    var test = result.test;
                    chi_calculated = test.calculated;
                    chi_accepted = test.critical;
                    var chi_table = test.intervals;
                    this.chi_calculated = chi_calculated;
                    this.chi_accepted = chi_accepted;
                    this.chi_table = chi_table;
                    console.table(chi_table);
                    console.log(chi_calculated);
                    console.log(chi_accepted);
                    console.log(interval_list);
                    console.log(data_list);
                })
                .catch(error => console.log('error', error));

            const data = interval_list.map((k, i) => ({ x: k, y: data_list[i] }));

            this.generateHistogram(data, min_value, max_value, interval_size);
        },

        async generateHistogram(data, min_value, max_value, interval_size) {
            const canvas = document.getElementById('histogram');
            this.chart = new Chart(canvas, {
                type: 'bar',
                data: {
                    datasets: [{
                        label: 'Número de Ocurrencias',
                        data: data,
                        borderWidth: 1,
                        barPercentage: 1,
                        categoryPercentage: 1,
                        borderRadius: 5,
                        parsing: false,
                    }]
                },
                options: {
                    animation: {
                        onComplete: this.enableButtons,
                    },
                    scales: {
                        x: {
                            min: min_value,
                            max: max_value,
                            type: 'linear',
                            offset: false,
                            grid: {
                                offset: false,
                            },
                            ticks: {
                                stepSize: interval_size,
                            },
                            title: {
                                display: true,
                                text: 'Número',
                                font: {
                                    size: 14,
                                }
                            }
                        },
                        y: {
                            beginAtZero: true,
                            title: {
                                display: true,
                                text: 'Ocurrencias',
                                font: {
                                    size: 14
                                }
                            }
                        }
                    },
                }
            });
        },
        formatNumber(number) {
            let formattedNumber = parseFloat(number.toFixed(3)).toString(); // Convert number to string and remove trailing zeros
            if (formattedNumber.includes('.')) {
                return formattedNumber; // Return with decimal part if present
            } else {
                return formattedNumber.replace('.', ''); // Remove decimal point if no decimal part
            }
        },
        generateRandomNumbers() {
            this.v$.$validate();
            if (this.v$.$error) {
                alert('Verifique los campos e intente nuevamente');
                console.log(this.v$.$errors);
                return;
            }
            this.disableButtons();
            var myHeaders = new Headers();
            myHeaders.append("Content-Type", "application/json");
            var url = "http://127.0.0.1:3000/api/generate";
            var requestOptions = {
                method: 'POST',
                headers: myHeaders,
                redirect: 'follow',
            };
            var data = {
                seed: this.seed,
                number: this.numGenerated,
            }
            var dist;
            if (this.distribution === 'normal') {
                data.distribution = "Normal";
                dist = {
                    mean: this.mean,
                    sd: this.sd,
                    algorithm: this.algorithm == "boxmuller" ? "BoxMuller" : "Convolution",
                    pair: null
                };
            } else if (this.distribution === 'uniform') {
                data.distribution = "Uniform";
                dist = {
                    lower: this.lowerLimit,
                    upper: this.upperLimit
                };
            } else if (this.distribution === 'exponential') {
                data.distribution = "Exponential";
                let paramlambda;
                if (this.lambda == "") {
                    paramlambda = 1/this.expmean;
                }
                else {
                    paramlambda = this.lambda;
                }
                dist = {
                    lambda: paramlambda
                };
            } else if (this.distribution === 'poisson') {
                data.distribution = "Poisson";
                dist = {
                    lambda: this.lambda
                };
            }
            data.data = dist;
            requestOptions.body = JSON.stringify(data);
            this.generatedNumbers = this.numGenerated;
            this.generated = false;
            this.last_dist = this.distribution;
            this.last_mean = this.mean;
            this.last_sd = this.sd;
            this.last_lower = this.lowerLimit;
            this.last_upper = this.upperLimit;
            this.last_lambda = this.lambda;
            fetch(url, requestOptions)
                .then(response => {
                    if (response.ok) {
                        this.generated = true;
                        this.regenIntervals();
                    }
                    else {
                        this.enableButtons();
                    }
                })
                .catch(error => {
                    console.log(error);
                });
        },
    },
};
</script>
