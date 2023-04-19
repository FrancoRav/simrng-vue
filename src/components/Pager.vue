<template>
    <div>
        <nav>
            <ul class="pages">
                <button class="cont" @click="changePage(currentPage-1)" :disabled="currentPage <= 1">≺</button>
                <span class="nr1">
                    <li class="pagenr nr1" v-if="totalPages > 0" :key="1" :class="{'active': 1 === currentPage}">
                        <a href="#" @click.prevent="changePage(1)">1</a>
                    </li>
                </span>
                <li class="pagenr nrcent" v-for="pageNumber in visiblePageNumbers" :key="pageNumber" :class="{'active': pageNumber === currentPage}">
                    <a href="#" @click.prevent="changePage(pageNumber)">{{ pageNumber }}</a>
                </li>
                <span class="nrlast">
                    <li class="pagenr last" v-if="totalPages > 1" :key="totalPages" :class="{'active': totalPages === currentPage}">
                        <a class="last" href="#" @click.prevent="changePage(totalPages)">{{ totalPages }}</a>
                    </li>
                </span>
                <button class="cont" @click="changePage(currentPage+1)" :disabled="currentPage >= totalPages">≻</button>
            </ul>
        </nav>
        <ul>
            <li v-for="item in displayedItems">{{ item }}</li>
        </ul>
    </div>
</template>

<style scoped>
.nr1 {
    margin-right: auto;
}
.nrlast {
    margin-left: auto;
    overflow: hidden;
}
.last {
    float: right;
}
.pages {
    display: flex;
    justify-content: center;
    align-items: center;
    list-style-type: none;
}
a {
    color: #1DA1F2;
}
.active a {
    color: gray;
}
span {
    white-space: nowrap;
}
div > ul {
    height: 300px;
}
</style>

<script>
export default {
    props: {
        totalPages: {
            type: Number,
            default: 0,
        }
    },
    data() {
        return {
            currentPage: 1,
            displayedItems: [],
        }
    },
    computed: {
        visiblePageNumbers() {
            const visiblePageRange = 2; // number of visible pages on each side of the current page
            const pageNumbers = [];
            let startPage = Math.max(2, this.currentPage - visiblePageRange);
            let endPage = Math.min(this.totalPages - 1, this.currentPage + visiblePageRange);

            for (let i = startPage; i <= endPage; i++) {
                pageNumbers.push(i);
            }

            return pageNumbers;
        }
    },
    watch: {
        currentPage() {
            this.fetchDisplayedItems();
        }
    },
    mounted() {
        this.fetchDisplayedItems();
    },
    methods: {
        fetchDisplayedItems() {
            // make API request to fetch displayed items based on current page number
            // update displayedItems data with the fetched items
            const url = `http://127.0.0.1:3000/api/page?page=${this.currentPage}`;
            // You can use your preferred HTTP library to make the API request, e.g. Axios, Fetch, etc.
            // Example with Fetch:
            var myHeaders = new Headers();
            myHeaders.append("Content-Type", "application/json");
            var requestOptions = {
                method: 'GET',
                headers: myHeaders,
                redirect: 'follow',
            };
            fetch(url, requestOptions)
                .then(response => response.json())
                .then(data => {
                    this.displayedItems = data;
                })
                .catch(error => {
                    console.error(`Failed to fetch displayed items: ${error}`);
                });
        },
        changePage(pageNumber) {
            this.currentPage = pageNumber;
        }
    }
}
</script>
