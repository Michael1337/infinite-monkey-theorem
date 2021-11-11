<template>
  <div>
    <h1>{{ msg }}</h1>
    <button v-on:click="click">click</button>
    <br>
    <button v-on:click="upgrade">upgrade auto clicker (costs {{ auto_clicker_update_cost }} points)</button>
    <br>
    Points: {{ points }}<br>
    Clicked {{ total_clicks }} times!<br>

    Latest letter: {{ latest_letter }} <br>
    Current stack: {{ current_word }} <br>
    <br>
    <div v-for="word in words_found" :key="'word_'+word[0]">
      {{ word[0] + ":" + word[1] }}
    </div>
    <br>
    <div v-for="letter in letters_found" :key="'letter_'+letter[0]">
      {{ letter[0] + ":" + letter[1] }}
    </div>
  </div>
</template>

<script>
import dict from '@/assets/words.txt';

const CURRENT_STACK_MAX_LENGTH = 20;
const MINIMUM_WORD_LENGTH = 3;
const MAXIMUM_WORD_LENGTH = 7;

export default {
  name: 'Game',
  data: function () {
    return {
      total_clicks: 0,
      points: 0,
      auto_click_level: 0,
      alphabet: ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', 'ü', 'ö', 'ä', 'ß'],
      dictionary: [],
      dictionary_regexp: '',
      latest_letter: '',
      current_stack: [],
      letters_found: new Map(),
      words_found: new Map(),
      auto_clicker: ''
    }
  },
  props: {
    msg: String
  },
  mounted() {
    this.initLettersFound();
    this.initDictionaryMap();
  },
  computed: {
    current_word: function () {
      return this.current_stack.join("");
    },
    auto_clicker_update_cost: function () {
      return Math.floor(10 * Math.pow(1.2, this.auto_click_level));
    }
  },
  methods: {
    initLettersFound: function () {
      this.alphabet.forEach(letter => this.letters_found.set(letter, 0));
    },
    initDictionaryMap: function () {
      this.dictionary = dict.split("\r\n").filter(word => word.length <= MAXIMUM_WORD_LENGTH && word.length >= MINIMUM_WORD_LENGTH);
      this.dictionary_regexp = "(" + this.dictionary.join("|").toLowerCase() + ")$";
    },
    updateAutoClicker: function () {
      clearInterval(this.auto_clicker);
      if (this.auto_click_level > 0) {
        this.auto_clicker = setInterval(() => {
          this.click();
        }, 1000 / this.auto_click_level);
      }
    },
    getRandomLetterFromAlphabet: function () {
      return this.alphabet[Math.floor(Math.random() * this.alphabet.length)];
    },
    updateCurrentStack: function (new_letter) {
      this.current_stack.push(new_letter);

      if (this.current_stack.length > CURRENT_STACK_MAX_LENGTH) {
        this.current_stack.shift();
      }
    },
    incrementLetterFound: function (latest_letter) {
      this.letters_found.set(latest_letter, this.letters_found.get(latest_letter) + 1);
    },
    currentStackIncludesAnyWord: function () {
      return new RegExp(this.dictionary_regexp).test(this.current_word);
    },
    awardPointsForLetter: function () {
      this.points++;
    },
    awardPointsForWord: function (previous_count, word) {
      // on first find, award 10 times as many points
      this.points += previous_count === 0 ? 10 * Math.pow(10, word.length - 2) : 10 * Math.pow(10, word.length - 3);
    },
    incrementWordFound: function (word, previous_count) {
      this.words_found.set(word, (previous_count + 1));
    },
    click: function () {
      this.total_clicks++;
      this.latest_letter = this.getRandomLetterFromAlphabet();
      this.awardPointsForLetter();

      this.updateCurrentStack(this.latest_letter);
      this.incrementLetterFound(this.latest_letter);

      if (this.currentStackIncludesAnyWord()) {
        const word = this.dictionary.filter(word => this.current_word.includes(word)).pop();

        if (typeof word !== "undefined") {
          const previous_count = this.words_found.get(word) ?? 0;
          this.incrementWordFound(word, previous_count);
          this.awardPointsForWord(previous_count, word);
        }
      }
    },
    upgrade: function () {
      if (this.points >= this.auto_clicker_update_cost) {
        this.points -= this.auto_clicker_update_cost;
        this.auto_click_level++;
        this.updateAutoClicker();
      }
    }
  }
}
</script>
