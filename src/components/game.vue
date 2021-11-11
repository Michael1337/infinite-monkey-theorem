<template>
  <div>
    <h1>{{ msg }}</h1>
    <button v-on:click="click">click</button>
    Points: {{ points }}<br>
    Clicked {{ clicks }} times!<br>

    Latest letter: {{ latest_letter }} <br>
    Current stack: {{ current_word }} <br>
    <br>
    <div v-for="word in words_found" :key="'word_'+word[0]">
      {{ word[0] + ":" + word[1] }}
    </div>
    <br>
    <div v-for="letter in alphabet_map" :key="'letter_'+letter[0]">
      {{ letter[0] + ":" + letter[1] }}
    </div>
  </div>
</template>

<script>
import dict from '../assets/words.txt';

export default {
  name: 'HelloWorld',
  data: function () {
    return {
      clicks: 0,
      current_stack: [],
      latest_letter: '',
      words_found: new Map(),
      points: 0,
      alphabet: ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', 'ü', 'ö', 'ä', 'ß', ' '],
      alphabet_map: new Map(),
      dictionary: []
    }
  },
  props: {
    msg: String
  },
  mounted() {
    this.initAlphabetMap();
    this.dictionary = dict.split("\r\n").filter(word => word.length <= 7 && word.length > 2);
    console.log(this.dictionary);
    setInterval(this.click, 1);
  },
  computed: {
    current_word: function () {
      return this.current_stack.join("");
    }
  },
  methods: {
    initAlphabetMap: function () {
      this.alphabet.forEach(letter => this.alphabet_map.set(letter, 0));
    },
    click: function () {
      ++this.clicks;
      this.latest_letter = this.alphabet[Math.floor(Math.random() * this.alphabet.length)];
      this.current_stack.push(this.latest_letter);
      if (this.current_stack.length > 20) {
        this.current_stack.shift();
      }
      this.alphabet_map.set(this.latest_letter, this.alphabet_map.get(this.latest_letter) + 1);

      const reg = "(" + this.dictionary.join("|").toLowerCase() + ")$";
      if (new RegExp(reg).test(this.current_word)) {
        //found a word
        const word = this.dictionary.filter(word => this.current_word.includes(word)).pop();

        if (typeof word !== "undefined") {
          const previous_count = this.words_found.get(word) ?? 0;
          this.words_found.set(word, (previous_count + 1));
          this.points += previous_count === 0 ? Math.pow(10, word.length - 1) : Math.pow(10, word.length - 2);
        }
      }
    }
  }
}
</script>
