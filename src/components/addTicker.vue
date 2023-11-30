<template>
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
                <div v-if="hints.length" class="flex max-w-xs border-b-2">
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
                <div v-if="isAlreadyAdded" class="max-w-xs ml-1 text-red-700">
                    Такой тикер уже добавлен
                </div>
                <div v-if="!isTockenExist" class="max-w-xs ml-1 text-red-700">
                    Такой валюты не существует
                </div>
            </div>
        </div>
        <add-button @click="add" class="my-4" />
    </section>
</template>

<script>
import addButton from "./addButton.vue";
import { getAllTickersTitle } from "../api";

export default {
    data() {
        return {
            ticker: "",
            hintsData: [],
        };
    },

    created: async function () {
        this.hintsData = await getAllTickersTitle();
    },

    components: {
        addButton,
    },

    props: {
        tickers: {
            type: Array,
            required: false,
            default: () => [],
        },
    },

    emits: {
        "add-ticker": (value) => typeof value === "string",
    },

    methods: {
        add() {
            if (this.isTockenExist && !this.isAlreadyAdded) {
                this.$emit("add-ticker", this.ticker);
                this.ticker = "";
            }
        },

        selectHint(h) {
            this.ticker = h;
            this.add();
        },
    },

    computed: {
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
};
</script>
