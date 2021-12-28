<template>
  <div id="app">
    <form class="filter" autocomplete="off" @submit.prevent>
      <input autocomplete="off" type="text" class="hidden">
      <label for="filter">
        Filter:
      </label>
      <div class="searchFieldBlock">
        <input type="search"
               class="searchField"
               name="filter"
               id="filter"
               v-model="filter"
               autocomplete="off">
        <span class="searchFieldTip">{{ filter }}
          <span class="searchFieldTipCounter">{{ filter ? `(${ filteredIcons.length })` : '' }}</span>
        </span>
      </div>
      <button v-for="(value, key, index) in allTypeFilters" :key="index" @click="toggleTypeFilter(value)" class="filterButton" :class="{ enabled: typeFilters.includes(value) }">
        {{ key }} <i
          class="check fad fa-check-circle"></i> <i class="checkNot fad fa-check-circle"></i></button>
      <span>{{ filteredIconsCount }}/{{ totalIconsCount }}</span>
      <input type="text" id="selection" value="" readonly  ref="selectionPlaceholder">
    </form>
    <article id="container" ref="container">
      <div
          v-for="(icon, id) in shownIcons"
          :key="id"
          class="iconContainer"
          :class="{ pro: icon.match(' pro') }"
          ref="iconContainer"
          @click="clipboardCopy(icon.replace(' pro', ''), id)"
          @animationend="disableCopyEffect">
        <i class="icon fa-5x"
           :class="icon.replace(' pro', '')"
           @animationend="disableCopyEffect"></i>
        <b>{{ icon.replace(' pro', '') }}</b>
        <span class="copiedMessage"
              @animationend="disableCopyEffect">
          <i class="fa-clipboard-check fad"></i>
          Copied!
        </span>
      </div>
    </article>
  </div>
</template>

<script>
// to build: $ vue build -d '../assets'

import allIcons from './data/icons.js';
import _ from 'lodash';

export default {
  name: 'app',
  components: {},
  data() {
    return {
      page: -1,
      perPage: 100,
      shownIcons: [],
      filteredIcons: [],
      allIcons: allIcons,
      textFilter: '',
      typeFilters: [],
      allTypeFilters: {
        'Pro': ' pro',
        'Solid': ' fas',
        'Regular': ' far',
        'Light': ' fal',
        'Duotone': ' fad',
        'Brands': ' fab',
      },
    };
  },
  watch: {
    filter: function() {
      this.switchFilters();
    },
    typeFilters: function() {
      this.switchFilters();
    },
    page: function() {
      this.pageArray();
    },
  },
  methods: {
    toggleTypeFilter(type) {
      this.typeFilters = _.xor(this.typeFilters, [type]);
    },
    switchFilters() {
      this.shownIcons = [];
      this.filterArray();
      this.page = 0;
      this.pageArray();
    },
    pageArray() {
      const sliceStart = this.page * this.perPage;
      const sliceEnd = (this.page + 1) * this.perPage;
      const nextPortion = this.filteredIcons.slice(sliceStart, sliceEnd);

      this.shownIcons = [...this.shownIcons, ...nextPortion];
    },
    filterArray() {
      const textFilteredIcons = this.filter ? [] : allIcons;
      if (this.textFilter) allIcons.map(icon => {
        if (icon.match(this.textFilter)) textFilteredIcons.push(icon);
      });

      const typeFilterExclude
          = this.typeFilters.length && this.typeFilters.length !== Object.values(this.allTypeFilters).length
            ? '(' + Object.values(this.allTypeFilters).filter(x => !this.typeFilters.includes(x)).join('|') + ')'
            : false;

      this.filteredIcons = (typeFilterExclude)
                           ? textFilteredIcons.filter(icon => !icon.match(typeFilterExclude))
                           : textFilteredIcons;

    },
    clipboardCopy(icon, elementId) {
      const iconContainer = this.$refs.iconContainer[elementId];

      const elIcon = iconContainer.querySelector('i.icon');
      elIcon.classList.add('animate__animated', 'animate__rubberBand');

      const elCopyMessage = iconContainer.querySelector('span.copiedMessage');
      elCopyMessage.classList.add('animate__animated', 'animate__fadeOutDown');

      const inputEl = this.$refs.selectionPlaceholder;

      inputEl.value = `<i class="${ icon.replace(' pro', '') }"></i>`;
      const selected = document.getSelection().rangeCount > 0
                       ? document.getSelection().getRangeAt(0)
                       : false;
      inputEl.select();
      document.execCommand('copy');

      if (selected) {
        document.getSelection().removeAllRanges();
        document.getSelection().addRange(selected);
      }
    },
    disableCopyEffect(event) {
      if(event.target.classList.contains('animate__animated')) {
        event.target.className = event.target.className.replace(/animate__\S+/g, '');
      }
    },
    showNextPortion() {
      this.page++;
    },
  },
  computed: {
    filter: {
      get() {
        return this.textFilter;
      },
      set: _.debounce(function(newValue) {
        this.textFilter = newValue;
      }, 150),
    },
    filteredIconsCount: function() {
      return this.filteredIcons.length;
    },
    totalIconsCount: function() {
      return this.allIcons.length;
    },
  },
  mounted() {
    document.title = 'FA like a Pro!';
    this.typeFilters = Object.values(this.allTypeFilters);
    this.filterArray();
    this.showNextPortion();
    const container = this.$refs.container;
    container.onscroll = () => {
      const bottomOfWindow = container.scrollTop + container.offsetHeight >= container.scrollHeight - 100;
      if (bottomOfWindow) this.showNextPortion();
    };
  },
};
</script>

<style>

body, html {
  margin: 0;
  padding: 0;
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
}

#app {
  padding: 1rem;
}

#selection {
  position: absolute;
  left: -9999px;
}

.hidden {
  display: none;
}

.filter {
  margin-bottom: 1rem;

}

.searchFieldBlock {
  display: inline-block;
  position: relative;
}

.searchField {
  background: transparent !important;
  font-size: 1rem;
  border-radius: 3px;
  border: 1px solid #777;
}

.searchFieldTip {
  display: inline-block;
  position: absolute;
  top: 0;
  left: 0;
  font-size: 1rem;
  color: transparent;
  border: 1px solid transparent;
  padding: 1px 2px;
}

.searchFieldTipCounter {
  color: #777 !important;
}

.filterButton {
  margin: 0.5rem;
}

.filterButton.enabled {
  color: darkgreen;
}

.filterButton .check {
  display: none;
}

.filterButton .checkNot {
  display: inline-block;
}

.filterButton.enabled .check {
  display: inline-block;
}

.filterButton.enabled .checkNot {
  display: none;
}

#container {
  display: grid;
  grid-template-columns: repeat(10, 1fr);
  grid-gap: 10px;
  align-items: center;
  box-sizing: border-box;
  max-height: 90vh;
  overflow: auto;
  padding: 2rem 0.5rem;
}

#container i.icon,
#container .iconContainer {
  align-self: center;
  text-align: center;
}

#container .iconContainer {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  padding: 5px;
  border: 1px dotted #777;
  color: #777;
  cursor: pointer;
}

#container .iconContainer.pro {
  border: 1px dotted cornflowerblue;
  color: cornflowerblue;
}

#container .iconContainer.pro:before {
  position: absolute;
  content: "pro";
  top: 5px;
  left: 5px;
  border-radius: 3px;
  padding: 5px;
  font-size: .5rem;
  border: 1px dotted coral;
}

#container .iconContainer b {
  background-color: #666;
  color: #fff;
  padding: 3px 5px;
  font-size: 0.75rem;
  margin-bottom: 10px;
}

#container .iconContainer .copiedMessage {
  visibility: hidden;
}

#container .iconContainer .copiedMessage.animate__animated {
  visibility: visible;
}

</style>
