<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <div class="container">
      <section>
        <div class="flex">
          <div class="max-w-xs">
            <div class="mt-1 relative rounded-md shadow-md">
              <input type="text" name="wallet" id="wallet" v-model="ticker" v-on:keydown.enter="add()"
                class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
                placeholder="Например BTC" />
            </div>
            <div v-if="isVariants" class="flex bg-white shadow-md p-1 rounded-md shadow-md flex-wrap">
              <span @click="ticker = coinName, add()" v-for="coinName, index in autoCompleteNames()" :key="index"
                class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer">
                {{ coinName }}
              </span>
            </div>
            <div v-if="isVisible" class="text-sm text-red-600">Такой тикер уже добавлен</div>
          </div>
        </div>
        <button type="button" @click="add()"
          class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500">
          <PlusSignIcon />
          Добавить
        </button>
        <div class="flex items-center justify-start w-64">
          <input v-model="filter" @input="page = 1" type="text" id="small-input" placeholder="Фильтр"
            class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md">
        </div>
        <div>
          <button @click="page = page - 1" v-if="page > 1"
            class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
            type="button">Назад</button>
          <button @click="page = page + 1" v-if="hasNextPage"
            class="my-4 ml-1 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
            type="button">Вперед</button>
        </div>
      </section>
      <div v-if="tickers.length">
        <hr class="w-full border-t border-gray-600 my-4" />
        <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <div v-for="t in paginatedTickers" :key="t.name" @click="select(t)"
            :class="selectedTicker == t ? 'border-4' : ''"
            class="bg-white overflow-hidden shadow rounded-lg border-purple-800 border-solid cursor-pointer">
            <div :class="formatPrice(t.price) == '-' ? 'bg-red-300' : 'bg-white'" class="px-4 py-5 sm:p-6 text-center">
              <dt class="text-sm font-medium text-gray-500 truncate">
                {{ t.name }} - USD
              </dt>
              <dd class="mt-1 text-3xl font-semibold text-gray-900">
                {{ formatPrice(t.price) }}
              </dd>
            </div>
            <div class="w-full border-t border-gray-200"></div>
            <button @click.stop="handleDelete(t)"
              class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none">
              <svg class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="#718096"
                aria-hidden="true">
                <path fill-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  clip-rule="evenodd"></path>
              </svg>Удалить
            </button>
          </div>
        </dl>
        <hr class="w-full border-t border-gray-600 my-4" />
      </div>
      <section v-if="selectedTicker" class="relative">
        <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
          {{ selectedTicker.name }} - {{ selectedTicker.price }}
        </h3>
        <div class="flex items-end border-gray-600 border-b border-l h-64" ref="graph">
          <div v-for="bar, i in normalazedGraph" :key="i" :style="{ height: `${bar}%` }"
            class="bg-purple-800 border w-10 h-24">
          </div>
        </div>
        <button @click="selectedTicker = null" type="button" class="absolute top-0 right-0">
          <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
            xmlns:svgjs="http://svgjs.com/svgjs" version="1.1" width="30" height="30" x="0" y="0"
            viewBox="0 0 511.76 511.76" style="enable-background:new 0 0 512 512" xml:space="preserve">
            <g>
              <path
                d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
                fill="#718096" data-original="#000000"></path>
            </g>
          </svg>
        </button>
      </section>
    </div>
  </div>
</template>

<script>
// [x] 6. наличие сосяний зависимых данных - критичность: 5+
// [x] 4. запросы напрямую внутри компонентов - критичность: 5
// [x] 2. при удалении остаётся подписка на подгрузку тикера - критичность: 5
// [h] 5. обработка ошибок API - критичность: 5
// [x] 3. количесто запросов - критичность: 4
// [x] 8. при удалении тикера не меняется localStorage - критичность: 4
// [x] 1. одинаковый код в watch (?) - критичность: 3
// [ ] 9. localStorage и анонимные вкладки - критичность: 3
// [x] 7. график ужасно выглядит, если будет много цен - критичность: 2
// [ ] 10. магические строки и числа (URL, 5000 мс задержка, ключ locallStorage кол-во на странице): 1

// параллельно
// [x] график сломан, если везде одинаковые значения
// [x] при удалении тикера остаётся выбор

//IMPORTS------------------------------------------------------------->

import { subscribeToTicker, unsubscribeFromTicker } from './api.js'
import PlusSignIcon from './components/PlusSignIcon.vue';

//EXPORT-------------------------------------------------------------->

export default {
  name: 'App',

  components: {
    PlusSignIcon
  },

  data() {
    return {
      ticker: '',
      filter: "",

      tickers: [],
      tickerNames: [],
      selectedTicker: null,

      graph: [],
      maxGraphElements: 1,

      isVisible: false,
      isVariants: false,

      coinsNames: [],

      page: 1,
    }
  },

  //MOUNTED------------------------------------------------------------>

  mounted() {
    return new Promise((resolve, reject) => {
      fetch('https://min-api.cryptocompare.com/data/all/coinlist?summary=true')
        .then(response => response.json())
        .then(data => {
          resolve(data), Array.from(Object.values(data.Data)).forEach((e) => {
            this.coinsNames.push(this.extractTextInsideBrackets(e.FullName));
          })
        })
        .catch(error => reject(error))
    })
  },

  beforeUnmount() {
    window.removeEventListener('resize', this.calculateMaxGraphElements())
  },
  //WATCH-------------------------------------------------------------->

  watch: {
    tickers() {
      localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers));
    },

    selectedTicker() {
      this.graph = [];
    },

    ticker(val) {
      if (val.length > 0) {
        this.isVariants = true;
      } else {
        this.isVariants = false;
      }
    },
    paginatedTickers() {
      if (this.paginatedTickers.length == 0 && this.page > 1) {
        this.page -= 1;
      }
    }
  },

  //CREATED----------------------------------------------------------->

  created() {
    window.addEventListener('resize', () => {
      this.calculateMaxGraphElements()
    })

    const tickersData = localStorage.getItem('cryptonomicon-list')

    if (tickersData) {
      this.tickers = JSON.parse(tickersData);
      this.tickers.forEach((ticker) => {
        subscribeToTicker(ticker.name, (newPrice) => this.updateTicker(ticker.name, newPrice));
      });
    }
  },

  //COMPUTED----------------------------------------------------------->

  computed: {
    startIndex() {
      return (this.page - 1) * 6;
    },

    endIndex() {
      return this.page * 6;
    },

    filteredTickers() {
      return this.tickers.filter(ticker => ticker.name.includes(this.filter.toUpperCase()));

    },

    paginatedTickers() {
      return this.filteredTickers.slice(this.startIndex, this.endIndex);
    },

    hasNextPage() {
      return this.filteredTickers.length > this.endIndex;
    },

    normalazedGraph() {
      const maxValue = Math.max(...this.graph);
      const minValue = Math.min(...this.graph);

      if (maxValue == minValue) {
        return this.graph.map(() => 50)
      }

      return this.graph.map(price => 3 + ((price - minValue) * 97) / (maxValue - minValue))
    },
  },

  //METHODS------------------------------------------------------------>

  methods: {

    calculateMaxGraphElements() {
      if (!this.$refs.graph) {
        return;
      }
      this.maxGraphElements = this.$refs.graph.clientWidth / 38
    },

    updateTicker(tickerName, price) {
      this.calculateMaxGraphElements()
      this.tickers.filter(t => t.name === tickerName).forEach(t => {
        if (t == this.selectedTicker) {
          this.graph.push(price)
        } while (this.graph.length > this.maxGraphElements) {
          this.graph.shift();
        }
        t.price = price
      })

    },


    formatPrice(price) {
      if (price === '-') {
        return price;
      }
      return price > 1 ? price.toFixed(2) : price.toPrecision(2);
    },

    autoCompleteNames() {
      return this.coinsNames.filter(name => name.includes(this.ticker.toUpperCase())).slice(0, 4);
    },

    extractTextInsideBrackets(inputStr) {
      let result = inputStr.match(/\(([^)]+)\)/g);
      result = result.map(text => text.replace(/\(|\)/g, ''));
      return result.join(' ');
    },

    add() {
      let currentTicker = {
        name: this.ticker.toUpperCase(),
        price: "-",
      }
      if (this.tickerNames.includes(currentTicker.name)) {
        this.isVisible = true;
        setTimeout(() => {
          this.isVisible = false;
        }, 3000);
      }
      this.tickers = [...this.tickers, currentTicker];
      this.ticker = "";
      this.filter = "";
      this.tickerNames.push(currentTicker.name);
      subscribeToTicker(currentTicker.name, (newPrice) => this.updateTicker(currentTicker.name, newPrice));

    },

    handleDelete(tickToRemove) {
      this.tickers = this.tickers.filter(t => t != tickToRemove);
      this.tickerNames = this.tickerNames.filter(t => t != tickToRemove.name);
      localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers));
      this.selectedTicker = null;
      unsubscribeFromTicker(tickToRemove.name);
    },

    select(tick) {
      this.selectedTicker = tick;
    },
  },
}
</script>
