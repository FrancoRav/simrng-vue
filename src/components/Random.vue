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
        <label>Semilla (x0):</label>
        <input type="number" v-model="seed">
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
        <span v-if="distribution == 'exponential' || distribution == 'poisson'">
            <br>
            <label>Lambda <span>λ</span>:</label>
            <input type="number" step="0.01" v-model="lambda">
        </span>
        <br>
        <button class="gen" @click="generateRandomNumbers" :disabled='inprogress'>Generar Números Aleatorios</button>
        <br>
        <!--<keep-alive>
            <RecycleScroller v-if="generatedNumbers.length > 0" class="scroller" :items="generatedNumbers" :item-size="30"
                v-slot="{ item }" :prerender="500">
                <div class="num">
                    {{ item }}
                </div>
            </RecycleScroller>
        </keep-alive>-->
        <!--        <span>
            <Pager class="page" v-if="generatedNumbers.length > 0" :items="generatedNumbers" :items-per-page="10" />
        </span>-->
        <!--<VirtualList v-if="numsObj.length > 0" :size="50" :remain="20" :dataComponent=functional :dataKey="'value'"
            :dataSources="numsObj">
        </VirtualList>-->
        <!--<div class="numlist" v-if="generatedNumbers.length > 0">
            <h2>Generated Numbers:</h2>
            <ul>
                <li v-for="number in generatedNumbers" :key="number">{{ Math.round(number * 100) / 100 }}</li>
            </ul>
        </div>-->
        <br>
        <label>Intervalos:</label>
        <div class="histgen">
            <input type="number" v-model="intervals">
            <button class="gen" @click="regenIntervals" :disabled='inprogress'>↻</button>
        </div>
        <br>
        <div class="canvas">
            <canvas id="histogram"></canvas>
        </div>
        <div class="chi">
            <p>El chi-cuadrado calculado fue de: {{ Number(chi_calculated).toFixed(2) }}, siendo necesario uno de
                {{ Number(chi_accepted).toFixed(2) }}</p>
        </div>
    </div>
</template>

<style>
.canvas {
    width: 100%;
}

.histgen {
    vertical-align: middle;
    display: flex;
    white-space: nowrap;
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

.page {
    width: 100%;
    height: 200px;
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
    max-height: 200px;
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
import 'chart.js'
import Pager from './Pager.vue'
export default {
    data() {
        return {
            isinprogress: false,
            distribution: 'normal',
            intervals: 5,
            seed: 0,
            numGenerated: 10,
            lowerLimit: 0,
            upperLimit: 1,
            chi_calculated: null,
            chi_accepted: null,
            mean: 10,
            sd: 1,
            algorithm: 'boxmuller',
            lambda: 5,
            generatedNumbers: [],
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
            }
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

        generateRandomNumbers() {
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
                dist = {
                    lambda: this.lambda
                };
            } else if (this.distribution === 'poisson') {
                data.distribution = "Poisson";
                dist = {
                    lambda: this.lambda
                };
            }
            data.data = dist;
            requestOptions.body = JSON.stringify(data);
            fetch(url, requestOptions)
                .then(response => {
                    if (response.ok) {
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
