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
        <button @click="generateRandomNumbers">Generate Random Numbers</button>
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
        <input type="number" v-model="intervals">
        <br>
        <div class="canvas">
            <canvas id="histogram"></canvas>
        </div>
    </div>
</template>

<style>
.canvas {
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
    margin-top: 10px;
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
</style>

<script>
import Chart from 'chart.js/auto';
import 'chart.js'
import Pager from './Pager.vue'
export default {
    data() {
        return {
            distribution: 'normal',
            intervals: 5,
            seed: 0,
            numGenerated: 10,
            lowerLimit: 0,
            upperLimit: 1,
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
    methods: {
        async generateHistogram() {
            if (this.chart) {
                this.chart.destroy();
            }

            var min_value = this.generatedNumbers.reduce((a, b) => {
                return Math.floor(Math.min(a, b));
            });
            var max_value = this.generatedNumbers.reduce((a, b) => {
                return Math.ceil(Math.max(a, b))
            });
            var reqdata = {
                nums: this.generatedNumbers,
                lower: min_value,
                upper: max_value,
                intervals: this.intervals,
            }
            var myHeaders = new Headers();
            myHeaders.append("Content-Type", "application/json");
            var url = "http://127.0.0.1:3000/api/histogram";
            var requestOptions = {
                method: 'POST',
                headers: myHeaders,
                redirect: 'follow',
            };
            requestOptions.body = JSON.stringify(reqdata);
            var interval_size = (max_value-min_value)/this.intervals;
            var interval_list = [];
            var data_list = [];
            await fetch(url, requestOptions)
                .then(response => response.json())
                .then(result => {
                    interval_list = result.x;
                    data_list = result.y;
                    console.log(interval_list);
                    console.log(data_list);
                })
                .catch(error => console.log('error', error));

            const data = interval_list.map((k, i) => ({ x: k, y: data_list[i] }));

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
            let generatedNumbers = [];
            var myHeaders = new Headers();
            myHeaders.append("Content-Type", "application/json");
            var url = "http://127.0.0.1:3000/api/";
            var requestOptions = {
                method: 'POST',
                headers: myHeaders,
                redirect: 'follow',
            };
            if (this.distribution === 'normal') {
                (this.algorithm == "boxmuller" ? url += "normal-bm" : url += "normal-conv");
                var data = {
                    seed: this.seed,
                    number: this.numGenerated,
                    mean: this.mean,
                    sd: this.sd
                };
            } else if (this.distribution === 'uniform') {
                url += "uniform";
                var data = {
                    seed: this.seed,
                    number: this.numGenerated,
                    lower: this.lowerLimit,
                    upper: this.upperLimit
                };
            } else if (this.distribution === 'exponential') {
                url += "exponential";
                var data = {
                    seed: this.seed,
                    number: this.numGenerated,
                    lambda: this.lambda
                };
            } else if (this.distribution === 'poisson') {
                url += "poisson";
                var data = {
                    seed: this.seed,
                    number: this.numGenerated,
                    lambda: this.lambda
                };
            }
            requestOptions.body = JSON.stringify(data);
            fetch(url, requestOptions)
                .then(response => response.json())
                .then(result => {
                    generatedNumbers = result;
                    this.generatedNumbers = generatedNumbers;
                    this.generateHistogram();
                })
                .catch(error => console.log('error', error));

        },
    },
};
</script>
