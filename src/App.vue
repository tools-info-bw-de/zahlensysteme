<script setup>
import "bootstrap/dist/css/bootstrap.min.css"
import "bootstrap"

import { reactive, onBeforeMount, computed } from "vue"

const state = reactive({
  inputBase: 2,
  outputBase: 10,
  input: "",
  output: "",
  inputInvalid: false,
  inputTooBig: false,
  explain: false,
  explainText: "",
})

let baseList = []

onBeforeMount(() => {
  createBaseList();
});

function createBaseList() {
  baseList = []
  for (let i = 2; i <= 16; i++) {
    let name = i
    if (i === 2) {
      name = "2 (Binär)"
    } else if (i === 8) {
      name = "8 (Oktal)"
    } else if (i === 10) {
      name = "10 (Dezimal)"
    } else if (i === 16) {
      name = "16 (Hexadezimal)"
    }
    baseList.push({
      name: name,
      base: i,
    })
  }
}

function isValidForBase(input, base) {
  var validChars = '0123456789ABCDEF'.slice(0, base);
  var regex = new RegExp('^[' + validChars + ']+$', 'i');
  return regex.test(input);
}

state.output = computed(() => {

  if (state.input === "") {
    state.inputInvalid = false
    return ""
  }

  if (state.inputBase > 16 || state.inputBase < 2 || state.outputBase > 16 || state.outputBase < 2) {
    state.inputInvalid = true
    return ""
  }

  if (!isValidForBase(state.input, state.inputBase)) {
    state.inputInvalid = true
    return ""
  }

  let decimal = parseInt(state.input, state.inputBase)  // parst bei großen Zahlen nicht richtig!
  let result = decimal.toString(state.outputBase)

  // reverse-check 
  let reverse = parseInt(result, state.outputBase)
  let reverseFinal = reverse.toString(state.inputBase)
  if (reverseFinal !== state.input) {
    state.inputInvalid = true
    state.inputTooBig = true
    return ""
  }
  state.inputTooBig = false

  if (result === "NaN") {
    state.inputInvalid = true
    return ""
  }
  state.inputInvalid = false
  return result
})


function switchInOut() {
  let newInput = state.output
  let newInputBase = state.outputBase
  state.outputBase = state.inputBase
  state.inputBase = newInputBase
  state.input = newInput
  createBaseList()
}

function getMaxLength(digit, base, power) {
  let result = (digit * base ** power).toString().length
  let calc1 = `${digit} * ${base}^${power}`.length
  let calc2 = `${digit} * ${base ** power}`.length
  return Math.max(result, calc1, calc2)
}

function explainTo10() {
  let result = "  "

  // 2 * 10^4 + 3 * 10^3 + ...
  for (let i = 0; i < state.input.length; i++) {
    let digit = parseInt(state.input[i], state.inputBase)
    let power = state.input.length - i - 1
    let maxLength = getMaxLength(digit, state.inputBase, power)
    let newSum = `${digit} * ${state.inputBase}^${power} + `
    result += newSum.padStart(maxLength + 3, " ")
  }

  result = result.slice(0, -3)
  result += "\n= "

  // 2 * 1000 + 3 * 100 + 4 * 10 + 5 * 1
  for (let i = 0; i < state.input.length; i++) {
    let digit = parseInt(state.input[i], state.inputBase)
    let power = state.input.length - i - 1
    let maxLength = getMaxLength(digit, state.inputBase, power)
    let newSum = `${digit} * ${state.inputBase ** power} + `
    result += newSum.padStart(maxLength + 3, " ")
  }

  result = result.slice(0, -3)
  result += "\n= "

  // 2000 + 300 + 40 + 5
  for (let i = 0; i < state.input.length; i++) {
    let digit = parseInt(state.input[i], state.inputBase)
    let power = state.input.length - i - 1
    let maxLength = getMaxLength(digit, state.inputBase, power)
    let newSum = `${digit * state.inputBase ** power} + `
    result += newSum.padStart(maxLength + 3, " ")
  }

  result = result.slice(0, -3)
  result += "\n= "

  let lineLength = result.split("\n")[0].length
  result += " ".repeat(lineLength - state.output.length - 2)

  result += state.output

  return result
}

function explainDecToBin() {
  let result = ""

  let decimal = parseInt(state.input, state.inputBase)
  let max1 = state.input.length
  let max2 = Math.floor(decimal / 2).toString().length
  console.log(max1, max2)

  while (decimal !== 0) {
    let rest = decimal % 2
    let newLine = " ".repeat(max1 - decimal.toString().length)
    newLine += decimal.toString()
    newLine += " : 2 = "
    console.log(Math.floor(decimal / 2), max2 - (Math.floor(decimal / 2).toString().length))
    newLine += " ".repeat(max2 - (Math.floor(decimal / 2).toString().length))
    newLine += `${Math.floor(decimal / 2)}`
    newLine += ` Rest ${rest}\n`
    result += newLine
    decimal = Math.floor(decimal / 2)
  }

  result += `\nResultat: ${state.output}`

  return result
}

state.explainText = computed(() => {
  if (state.input === "" || state.inputInvalid) {
    return ""
  }

  // irgendwas zu dezimal
  if (state.outputBase === 10) {
    return explainTo10()
  }

  // dezimal zu binär
  if (state.inputBase === 10 && state.outputBase === 2) {
    return explainDecToBin()
  }

  // dezimal zu oktal?
  // dezimal zu hexadezimal
  // binär zu hexadezimal
  // hexadezimal zu binär
  // binär zu hexa?? und umgekehrt?
  // oktal zu hexa?? und umgekehrt?

  // und sonst: erst zu dezimal und dann weiter
})

</script>

<template>
  <h1 class="mb-4">Zahlensystem-Konverter</h1>
  <div class="row">
    <div class="ioBox col-md-12 col-lg-5 d-flex flex-column">
      <div class="labelInputOutput">Eingabe</div>
      <div class="d-flex flex-row">
        <div class="form-floating">
          <select class="form-select" id="inputList" v-model="state.inputBase">
            <option v-for="b in baseList" :value="b.base">{{ b.name }}</option>
          </select>
          <label for="inputList">Zahlensystem</label>
        </div>

        <div class="ms-2 flex-fill d-flex flex-column">
          <div class="form-floating">
            <input type="text" class="form-control" :class="{ redBorder: state.inputInvalid }" id="input"
              placeholder="Eingabe" autofocus v-model="state.input">
            <label for="input">Eingabe</label>
          </div>
          <div class="inputInvalidMessage" :class="{ invisible: !state.inputInvalid }">
            <div v-if="state.inputTooBig">Zahl zu groß (Überlauf)</div>
            <div v-else>Eingabe fehlerhaft!</div>
          </div>
        </div>
      </div>

    </div>
    <div class="col-md-12 col-lg-2 d-flex align-items-center justify-content-center">
      <button type="button" class="btn btn-primary my-3 d-flex align-items-center" @click.prevent="switchInOut()">
        <div class="me-2">Vertauschen</div>
        <img src="./assets/arrow-right-arrow-left-solid.svg" width="30" height="30">
      </button>
    </div>
    <div class="ioBox col-sm-12 col-lg-5 d-flex flex-column">
      <div class="labelInputOutput">Ausgabe</div>
      <div class="d-flex flex-row">
        <div class="form-floating">
          <select class="form-select" id="outputList" v-model="state.outputBase">
            <option v-for="b in baseList" :value="b.base">{{ b.name }}</option>
          </select>
          <label for="outputList">Zahlensystem</label>
        </div>

        <div class="ms-2 flex-fill d-flex flex-column">
          <textarea class="form-control output" :value="state.output" cols="30" rows="2" readonly></textarea>
        </div>
      </div>
    </div>
  </div>
  <div class="row mt-5 mb-2">
    <div class="col">
      <div class="form-check form-switch d-flex align-items-center">
        <input class="form-check-input ms-0" v-model="state.explain" type="checkbox" role="switch" id="explainSwitch">
        <label class="form-check-label ms-2" for="explainSwitch">Umrechnung erklären</label>
      </div>
    </div>
  </div>
  <div v-if="state.explain" class="row">
    <div class="col">
      <textarea class="form-control explainTextarea" :value="state.explainText" cols="30" rows="10" readonly></textarea>
    </div>
  </div>
</template>

<style scoped>
.explainTextarea {
  font-family: monospace;
  white-space: pre;
  background-color: #f0f0f0;
}

.invisible {
  visibility: hidden;
}

.redBorder {
  border-color: #842029;
  border-width: 2px;
}

.inputInvalidMessage {
  align-self: flex-start;
  background-color: #f8d7da;
  color: #842029;
  padding: 5px;
  border-radius: 5px;
}

.inputInvalid {
  border-color: #842029;
}

input,
select {
  border-color: gray;
}

input {
  font-size: 1.5rem;
}

.output {
  font-size: 1.5rem;
}

.ioBox {
  background-color: rgb(228, 228, 228);
  border-radius: 15px;
  padding: 10px;
}

.labelInputOutput {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 10px;
}
</style>
