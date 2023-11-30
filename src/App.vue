<template>
    <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
        <div class="container">
            <div class="w-full my-4"></div>
            <section>
                <div class="flex">
                    <div class="max-w-xs">
                        <label
                            for="wallet"
                            class="block text-sm font-medium text-gray-700"
                            >Тикер</label
                        >
                        <div class="mt-1 relative rounded-md shadow-md">
                            <input
                                v-model="ticker"
                                @keydown.enter="add"
                                type="text"
                                name="wallet"
                                id="wallet"
                                class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
                                placeholder="Например DOGE"
                            />
                        </div>
                        <div
                            v-if="hints.length"
                            class="flex max-w-xs border-b-2"
                        >
                            <div
                                v-for="h in hints"
                                :key="h"
                                class="pr-2 pl-2 my-2 mx-1 uppercase border-2 rounded-xl bg-gray-200 cursor-pointer"
                                @click="selectHint(h)"
                            >
                                {{ h }}
                            </div>
                            <div></div>
                        </div>
                        <div
                            v-if="isAlreadyAdded"
                            class="max-w-xs ml-1 text-red-700"
                        >
                            Такой тикер уже добавлен
                        </div>
                        <div
                            v-if="!isTockenExist"
                            class="max-w-xs ml-1 text-red-700"
                        >
                            Такой валюты не существует
                        </div>
                    </div>
                </div>
                <button
                    @click="add"
                    type="button"
                    class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
                >
                    <!-- Heroicon name: solid/mail -->
                    <svg
                        class="-ml-0.5 mr-2 h-6 w-6"
                        xmlns="http://www.w3.org/2000/svg"
                        width="30"
                        height="30"
                        viewBox="0 0 24 24"
                        fill="#ffffff"
                    >
                        <path
                            d="M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"
                        ></path>
                    </svg>
                    Добавить
                </button>
            </section>

            <template v-if="tickers.length">
                <hr class="w-full border-t border-gray-600 my-4" />
                <div>
                    <button
                        :disabled="page <= 1"
                        class="my-4 mx-2 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500 disabled:bg-gray-300"
                        @click="page = page - 1"
                    >
                        Назад
                    </button>
                    <button
                        :disabled="!hasNextPage"
                        class="my-4 mx-2 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500 disabled:bg-gray-300"
                        @click="page = page + 1"
                    >
                        Вперед
                    </button>
                    <div>
                        Фильтр:
                        <input
                            v-model="filter"
                            class="pr-10 ml-2 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
                        />
                    </div>
                </div>
                <hr class="w-full border-t border-gray-600 my-4" />
                <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
                    <div
                        v-for="t in paginatedTickers"
                        :key="t.name"
                        @click="selectTicker(t)"
                        :class="{
                            'border-4':
                                selectTicker && selectedTicker.name === t.name,
                        }"
                        class="bg-white overflow-hidden shadow rounded-lg border-purple-800 border-solid cursor-pointer"
                    >
                        <div class="px-4 py-5 sm:p-6 text-center">
                            <dt
                                class="text-sm font-medium text-gray-500 truncate"
                            >
                                {{ t.name }} - USD
                            </dt>
                            <dd
                                class="mt-1 text-3xl font-semibold text-gray-900"
                            >
                                {{ formatPrice(t.price) }}
                            </dd>
                        </div>
                        <div class="w-full border-t border-gray-200"></div>
                        <button
                            @click.stop="handleDelete(t)"
                            class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none"
                        >
                            <svg
                                class="h-5 w-5"
                                xmlns="http://www.w3.org/2000/svg"
                                viewBox="0 0 20 20"
                                fill="#718096"
                                aria-hidden="true"
                            >
                                <path
                                    fill-rule="evenodd"
                                    d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                                    clip-rule="evenodd"
                                ></path></svg
                            >Удалить
                        </button>
                    </div>
                </dl>
                <hr class="w-full border-t border-gray-600 my-4" />
            </template>
            <section v-if="selectedTicker" ref="graph" class="relative">
                <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
                    {{ selectedTicker.name }} - USD
                </h3>
                <div
                    class="flex items-end border-gray-600 border-b border-l h-64"
                >
                    <div
                        v-for="(bar, idx) in normalizedGraph"
                        :key="idx"
                        :style="{ height: `${bar}%` }"
                        ref="graphElement"
                        class="bg-purple-800 border w-10 flex-shrink-0"
                    ></div>
                </div>
                <button
                    @click="selectedTicker = false"
                    type="button"
                    class="absolute top-0 right-0"
                >
                    <svg
                        xmlns="http://www.w3.org/2000/svg"
                        xmlns:xlink="http://www.w3.org/1999/xlink"
                        xmlns:svgjs="http://svgjs.com/svgjs"
                        version="1.1"
                        width="30"
                        height="30"
                        x="0"
                        y="0"
                        viewBox="0 0 511.76 511.76"
                        style="enable-background: new 0 0 512 512"
                        xml:space="preserve"
                    >
                        <g>
                            <path
                                d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
                                fill="#718096"
                                data-original="#000000"
                            ></path>
                        </g>
                    </svg>
                </button>
            </section>
        </div>
    </div>
</template>

<script>
// [x] 6. Наличие в состоянии ЗАВИСИМЫХ ДАННЫХ | Критичность: 5+
// [x] 4. Запросы напрямую внутри компонента (???) | Критичность: 5
// [x] 2. При удалении остается подписка на загрузку тикера | Критичность: 5
// [x] 5. Обработка ошибок API | Критичность: 5
// [x] 3. Количество запросов | Критичность: 4
// [x] 8. При удалении тикера не изменяется localStorage | Критичность: 4
// [x] 1. Одинаковый код в watch | Критичность: 3
// [ ] 9. localStorage и анонимные вкладки | Критичность: 3
// [ ] 7. График ужасно выглядит если будет много цен | Критичность: 2
// [ ] 10. Магические строки и числа (URL, 5000 миллисекунд задержки, ключ локал стораджа, количество на странице) |  Критичность: 1

// Параллельно
// [x] График сломан если везде одинаковые значения
// [x] При удалении тикера остается выбор

import {
    subscribeToTicker,
    unsubscribeFromTicker,
    getAllTickersTitle,
} from "./api";

export default {
    name: "App",

    data() {
        return {
            ticker: "",
            filter: "",

            tickers: [],
            selectedTicker: false,

            graph: [],
            //graphElementWidth: 40,
            page: 1,

            hintsData: [],
        };
    },

    created: async function () {
        this.hintsData = await getAllTickersTitle();

        const windowData = Object.fromEntries(
            new URL(window.location).searchParams.entries()
        );

        const VALID_KEYS = ["filter", "page"];

        VALID_KEYS.forEach((key) => {
            if (windowData[key]) {
                this[key] = windowData[key];
            }
        });

        const tickersData = localStorage.getItem("cryptonomicon-list");

        if (tickersData) {
            this.tickers = JSON.parse(tickersData);
            this.tickers.forEach((ticker) => {
                subscribeToTicker(ticker.name, (newPrice) =>
                    this.updateTicker(ticker.name, newPrice)
                );
            });
        }
    },

    computed: {
        startIndex() {
            return (this.page - 1) * 6;
        },

        endIndex() {
            return this.page * 6;
        },

        filteredTickers() {
            return this.tickers.filter((ticker) =>
                ticker.name.includes(this.filter)
            );
        },

        paginatedTickers() {
            return this.filteredTickers.slice(this.startIndex, this.endIndex);
        },

        hasNextPage() {
            return this.filteredTickers.length > this.endIndex;
        },

        normalizedGraph() {
            const maxValue = Math.max(...this.graph);
            const minValue = Math.min(...this.graph);

            if (maxValue === minValue) {
                return this.graph.map(() => 50);
            }

            return this.graph.map(
                (price) => 5 + ((price - minValue) * 95) / (maxValue - minValue)
            );
        },

        pageStateOptions() {
            return {
                filter: this.filter,
                page: this.page,
            };
        },

        isAlreadyAdded() {
            return this.tickers.find(
                (t) => t.name === this.ticker.toUpperCase()
            );
        },

        hints() {
            let result = [];

            if (this.ticker) {
                for (let hint of this.hintsData) {
                    if (
                        hint
                            .toLowerCase()
                            .includes(this.ticker.toLowerCase()) &&
                        !this.tickers.find((t) => t.name === hint)
                    )
                        result.push(hint);
                    if (result.length >= 4) break;
                }
            }

            return result;
        },

        isTockenExist() {
            return this.ticker === "" || this.hints.length;
        },
    },

    methods: {
        updateTicker(tickerName, price) {
            let findedTickerIndex = this.tickers.findIndex(
                (t) => t.name === tickerName
            );
            if (
                this.tickers[findedTickerIndex].name ===
                this.selectedTicker.name
            ) {
                console.log(this.graph);
                this.graph = [...this.graph, price];
            }
            this.tickers[findedTickerIndex] = {
                name: tickerName,
                price: price,
            };
        },

        formatPrice(price) {
            if (price === "-") {
                return price;
            }
            return price > 1 ? price.toFixed(2) : price.toPrecision(2);
        },

        add() {
            const newTicker = {
                name: this.ticker.toUpperCase(),
                price: "-",
            };

            if (this.isTockenExist && !this.isAlreadyAdded) {
                this.tickers = [...this.tickers, newTicker];
                this.ticker = "";
                this.filter = "";
                this.hints = [];
                subscribeToTicker(newTicker.name, (newPrice) =>
                    this.updateTicker(newTicker.name, newPrice)
                );
            }
        },

        selectTicker(ticker) {
            this.selectedTicker = ticker;
        },

        handleDelete(tickerToRemove) {
            this.tickers = this.tickers.filter((t) => t !== tickerToRemove);
            if (
                this.selectedTicker &&
                this.selectedTicker.name === tickerToRemove.name
            ) {
                this.selectedTicker = false;
            }
            unsubscribeFromTicker(tickerToRemove.name);
        },

        selectHint(h) {
            this.ticker = h;
            this.add();
        },

        calcMaxGraphElements() {
            if (this.$refs.graph && this.$refs.graphElement) {
                return (
                    Math.floor(
                        this.$refs.graph.clientWidth /
                            this.$refs.graphElement[0].clientWidth
                    ) - 1
                );
            } else return 1;
        },
    },

    watch: {
        selectedTicker() {
            this.graph = [];
        },

        tickers() {
            localStorage.setItem(
                "cryptonomicon-list",
                JSON.stringify(this.tickers)
            );
        },

        paginatedTickers() {
            if (this.paginatedTickers.length === 0 && this.page > 1) {
                this.page -= 1;
            }
        },

        filter() {
            this.page = 1;
        },

        pageStateOptions(value) {
            window.history.pushState(
                null,
                document.title,
                `${window.location.pathname}?filter=${value.filter}&page=${value.page}`
            );
        },

        graph() {
            if (this.graph.length > this.calcMaxGraphElements())
                this.graph = this.graph.slice(
                    this.graph.length - this.calcMaxGraphElements()
                );
        },
    },
};
</script>