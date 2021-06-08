<template>
  <div id="home">
    <div id="controller">
      <select v-model="selectedPitch">
        <option value="-1">N</option>
        <option value="0">Ab</option>
        <option value="1">A</option>
        <option value="2">Bb</option>
        <option value="3">B</option>
        <option value="4">C</option>
        <option value="5">Db</option>
        <option value="6">D</option>
        <option value="7">Eb</option>
        <option value="8">E</option>
        <option value="9">F</option>
        <option value="10">Gb</option>
        <option value="11">G</option>
      </select>
      <div class="type">
        <input type="radio" id="guitar" value="G" v-model="type">
        <label for="guitar">Guitar</label>
        <input type="radio" id="bass" value="B" v-model="type">
        <label for="bass">Bass</label>
      </div>
      <span class="label">시험 간격(초)</span>
      <select v-model="testInterval" :disabled="testMode">
        <option value="1">1</option>
        <option value="3">3</option>
        <option value="5">5</option>
      </select>
      <div class="buttons">
        <div id="test-button" :class="{test: testMode}" @click="test">Test</div>
      </div>
    </div>
    <div id="fingerboard">
      <div class="string"
           :class="{select: selectedString === string}"
           v-for="string in 6" v-show="type === 'G' || (type === 'B' && string > 2)"
           @click="selectString(string)">
        <div class="fret" :class="{test: checkTest(string, fret) && testMode, select: checkPitch((stringPitch[string] + fret - 1) % 12)}" @click="testClick" v-for="fret in 13">
          <span class="answer"
                v-show="(checkTest(string, fret) && showAnswer) || !testMode" >
            {{pitch[(stringPitch[string] + fret - 1) % 12]}}
          </span>
        </div>
      </div>
    </div>
    <div id="guide">
      <div class="dot-area" v-for="dot in 13">
        <div v-if="dot > 2 && dot <= 10 && dot%2 === 0" class="dot"></div>
        <div v-if="dot === 13" class="dot"></div>
        <div v-if="dot === 13" class="dot"></div>
      </div>
    </div>
  </div>
</template>

<script>
import {Vue, Component} from 'vue-property-decorator'

@Component
export default class Home extends Vue{
  pitch = ['Ab', 'A', 'Bb', 'B', 'C', 'Db', 'D', 'Eb', 'E', 'F', 'Gb', 'G'];
  stringPitch = [0, 8, 3, 11, 6, 1, 8];
  selectedPitch = "-1";
  selectedString = 0;
  type = "G";

  testMode = false;
  testString = 0;
  testFret = 0;
  recentFret = new Queue();
  timerId = 0;
  testInterval = "3";
  showAnswer = false;

  selectString(input) {
    if (this.testMode) return;
    this.selectedString = this.selectedString === input ? 0 : input;
  }

  checkPitch(input) {
    return input === parseInt(this.selectedPitch)
  }

  checkTest(string, fret) {
    return !this.testMode || (this.testString === string && this.testFret === fret)
  }

  test () {
    this.testMode = !this.testMode;

    if (this.testMode) {
      this.testString = 0;
      this.testFret = 0;
      this.doTest();

      this.timerId = setInterval(() => {
        this.doTest();
      }, parseInt(this.testInterval) * 1000)
    } else {
      this.showAnswer = false;
      clearInterval(this.timerId)
    }
  }

  doTest () {
    this.showAnswer = false;
    this.testString = parseInt(this.selectedString !== 0 ? this.selectedString : this.getRandom(this.type === "G" ? 1 : 3, 7));
    this.testFret = this.getTestFret();
  }

  testClick () {
    if (this.testMode) {
      this.showAnswer = true;
    }
  }

  getRandom(min, max) {
    return Math.random() * (max - min) + min;
  }

  getTestFret() {
    let num;

    if(this.recentFret.size() > 5) {
      this.recentFret.dequeue();
    }

    while (true) {
      num = parseInt(this.getRandom(1, 13));
      if (!this.recentFret.include(num)) {
        this.recentFret.enqueue(num);
        break;
      }
    }

    this.recentFret.monitor();
    return num;
  }
}

class Queue {
  constructor() {
    this._arr = [];
  }
  enqueue(item) {
    this._arr.push(item);
  }
  dequeue() {
    return this._arr.shift();
  }
  include(value) {
    return this._arr.includes(value);
  }
  size() {
    return this._arr.length;
  }
  monitor() {
    console.log(this._arr);
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
#home {
  #controller {
    display: flex;
    flex-direction: row;
    align-items: center;
    height: 50px;
    width: 790px;

    select {
      height: 30px;
      width: 50px;
      border-color: gray;
      border-radius: 3px;
    }

    .type {
      display: flex;
      justify-content: center;
      align-items: center;
      margin-left: 20px;
    }

    .label {
      margin-left: 30px;
      margin-right: 10px;
    }

    #test-button {
      margin-left: 20px;
      line-height: 28px;
      height: 28px;
      width: 50px;
      border: solid 1px gray;
      border-radius: 3px;
      cursor: pointer;

      &.test {
        background-color: gray;
        color: white;
      }
    }
  }

  #fingerboard {
    display: flex;
    flex-direction: column;
    width: 650px;
    border-bottom: solid 1px black;

    .string {
      width: 650px;
      display: flex;
      flex-direction: row;
      border-top: solid 1px black;

      &.select {
        font-weight: bold;
      }

      .fret {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 49px;
        height: 30px;
        border-right: solid 1px black;
        background-color: wheat;

        &.test {
          background-color: lightslategray !important;
          cursor: pointer;
          color: white;
          font-weight: bold;
        }

        &.select {
          background-color: gold !important;
        }

        &:first-child {
          width: 48px;
          border-right-width: 2px;
          background-color: white;
        }
      }
    }
  }

  #guide {
    display: flex;
    flex-direction: row;

    .dot-area {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 50px;
      height: 10px;

      .dot {
        width: 8px;
        height: 8px;
        background-color: black;
        border-radius: 8px;
      }
    }
  }
}
</style>
