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
        <keep-alive>
            <RecycleScroller v-if="generatedNumbers.length > 0" class="scroller" :items="generatedNumbers" :item-size="10"
                v-slot="{ item }" :prerender="100">
                <div class="num">
                    {{ item }}
                </div>
            </RecycleScroller>
        </keep-alive>
        <!--<VirtualList v-if="numsObj.length > 0" :size="50" :remain="20" :dataComponent=functional :dataKey="'value'"
            :dataSources="numsObj">
        </VirtualList>-->
        <div class="numlist" v-if="generatedNumbers.length > 0">
            <h2>Generated Numbers:</h2>
            <ul>
                <li v-for="number in generatedNumbers" :key="number">{{ number }}</li>
            </ul>
        </div>
    </div>
</template>

<style>
.form {
    display: flex;
    overflow-x: hidden;
    overflow: hidden;
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
import { RecycleScroller } from 'vue-virtual-scroller'
export default {
    data() {
        return {
            distribution: 'normal',
            seed: 0,
            numGenerated: 10,
            lowerLimit: 0,
            upperLimit: 1,
            generatedNumbers: [],
            numsObj: [],
        };
    },
    components: {
        RecycleScroller
    },
    methods: {
        generateRandomNumbers() {
            // Use the values of distribution, seed, numGenerated, lowerLimit, and upperLimit
            // to generate an array of random numbers using the appropriate distribution.

            // For example, to generate an array of normally-distributed random numbers:
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
                    console.log(generatedNumbers);
                    this.generatedNumbers = generatedNumbers;
                    this.numsObj = generatedNumbers.map((num, index) => ({ id: index, value: num }));
                })
                .catch(error => console.log('error', error));
        },
    },
};
</script>
