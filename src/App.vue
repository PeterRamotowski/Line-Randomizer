<template>
  <div id="wrapper">
    <h1>Random line selection</h1>
    <div id="content">

      <label for="cycles" id="cycles">
        Set time:
        <input type="text" name="cycles" v-model="cycles">
        s
      </label>

      <textarea id="txt" placeholder="paste multiline content" v-model.trim="source" @input="sourceChange" @keyup.enter="checkUniques" @paste="resetOnPaste"></textarea>

      <div id="found" v-if="found > 0">Found {{ found }} lines</div>

      <button type="button" id="start" @click="initCycle" :class="{ hide: isCounting }" :disabled="!source">START</button>
      <button type="button" id="stop" @click="cancelCycle" :class="{ hide: !isCounting }">{{ counter }}END</button>

      <div id="result" :class="{ finished: !isCounting }">{{ randomLine }}</div>

      <button type="button" id="clear" @click="clearResults" v-if="!isCounting && finished">CLEAR RESULTS</button>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return { 
      source: '',
      randomLine: '',
      counter: '',
      found: 0,
      isCounting: false
    }
  },
  methods: {
    getRandomLine() {
      const lines = this.source.split("\n");
      const line = Math.floor(Math.random() * lines.length);
      this.randomLine = lines[line];
    },
    initCycle() {
      this.currentCounter = this.cycles;
      this.displayCounter();
      this.toggleButtons();
      this.checkUniques();
      this.intervalId = window.setInterval(this.getRandomLine, 100);
      this.cyclesId = window.setInterval(this.displayCounter, 1000);
      this.timeoutId = window.setTimeout(this.cancelCycle, this.cycles * 1000);
    },
    displayCounter() {
      this.counter = this.currentCounter-- + 's to ';
    },
    cancelCycle() {
      this.counter = '';
      this.finished = true;
      this.toggleButtons();
      clearInterval(this.intervalId);
      clearTimeout(this.timeoutId);
      clearInterval(this.cyclesId);
    },
    sourceChange() {
      this.finished = false;
      // clean results when emptied source input 
      if (!this.source) {
        this.randomLine = '';
        this.found = '';
      }
      else {
        let lines = this.source.split("\n");
        lines = lines.filter(row => row != ''); // remove empty lines
        this.source = lines.join("\n");
      }
    },
    checkUniques() {
      let lines = this.source.split("\n");
      lines = lines.filter((row, index) => lines.indexOf(row) == index);
      this.source = lines.join("\n");
      this.found = lines.length;
    },
    resetOnPaste() {
      this.randomLine = '';
      this.found = '';
      setTimeout( () => this.checkUniques(), 100 );
    },
    clearResults() {
      this.source = '';
      this.randomLine = '';
      this.found = '';
      this.finished = false;
    },
    toggleButtons() {
      this.isCounting = !this.isCounting;
    },
  },
  created() {
    this.intervalId;
    this.timeoutId;
    this.cyclesId;
    this.cycles = 3;
    this.currentCounter = 0;
    this.finished = false;
  }
}
</script>

<style lang="scss">
@mixin border-radius {
  border-radius: 5px;
}
@mixin font-family {
  font-family: Helvetica, Arial, sans-serif;
}
@mixin form-focus {
  outline: none;
  border-color: #000;
}
@mixin form-inputs {
  @include font-family;
  padding: 5px;
  border: 1px solid #ccc;
  @include border-radius;
  &:focus {
    @include form-focus;
    &::placeholder {
      opacity: 0;
    }
  }
}
html {
  box-sizing: border-box;
  * {
    box-sizing: inherit;
  }
}
body {
  @include font-family;
  background: #20262E;
  margin: 0;
}
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin: 2em;
}
.hide {
  display: none;
}
#content {
  display: flex;
  flex-direction: column;
  color: #2c3e50;
  background: #fff;
  @include border-radius;
  padding: 20px;
}
h1 {
  color: #fff;
}
#cycles {
  align-self: end;
  input {
    @include form-inputs;
    display: inline-block;
    font-size: 1.25em;
    width: 2em;
    text-align: center;
  }
}
textarea {
  @include form-inputs;
  width: 100%;
  height: 15rem;
  margin-top: 1rem;
}
button {
  align-self: center;
  background: #cc0000;
  color: #fff;
  font-size: 1.5em;
  font-weight: bold;
  margin-top: 1rem;
  padding: 5px 10px;
  @include border-radius;
  border: none;
  cursor: pointer;
  &:disabled {
    background: #ccc;
    cursor: default;
  }
}
#found {
  margin-top: 1rem;
}
#result:not(:empty) {
  align-self: center;
  white-space: pre-line;
  margin-top: 2rem;
  padding: 10px;
  @include border-radius;
  border: 1px solid #ccc;
  &.finished {
    &::before {
      display: block;
      content: 'result: ';
      font-size: 1rem;
      font-weight: normal;
      margin-bottom: .5rem;
    }
    font-size: 1.5em;
    font-weight: bold;
  }
}
</style>
