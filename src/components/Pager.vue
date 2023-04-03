<template>
    <div>
        <nav>
            <ul>
                <li class="pagenr" v-for="pageNumber in totalPages" :key="pageNumber">
                    <a href="#" @click.prevent="changePage(pageNumber)">{{ pageNumber }}</a>
                </li>
            </ul>
        </nav>
        <ul>
            <li v-for="item in displayedItems">{{ item }}</li>
        </ul>
    </div>
</template>

<script>
export default {
    props: {
        items: {
            type: Array,
            required: true
        },
        itemsPerPage: {
            type: Number,
            default: 10
        }
    },
    data() {
        return {
            currentPage: 1
        }
    },
    computed: {
        displayedItems() {
            const startIndex = (this.currentPage - 1) * this.itemsPerPage;
            const endIndex = startIndex + this.itemsPerPage;
            return this.items.slice(startIndex, endIndex);
        },
        totalPages() {
            return Math.ceil(this.items.length / this.itemsPerPage);
        }
    },
    methods: {
        changePage(pageNumber) {
            this.currentPage = pageNumber;
        }
    }
}
</script>
