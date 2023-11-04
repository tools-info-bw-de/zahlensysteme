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

    let decimal = parseInt(state.input, state.inputBase)
    let result = decimal.toString(state.outputBase)

    console.log(decimal, result)

    if (result === "NaN") {
      state.inputInvalid = true
      return ""
    }
    state.inputInvalid = false
    return result
  })
};

function switchInOut() {
  let newInput = state.output
  let newInputBase = state.outputBase
  state.outputBase = state.inputBase
  state.inputBase = newInputBase
  state.input = newInput
  createBaseList()
}

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
            <input type="text" class="form-control" id="input" placeholder="Eingabe" autofocus v-model="state.input">
            <label for="input">Eingabe</label>
          </div>
          <div v-if="state.inputInvalid" class="inputInvalidMessage">
            Eingabe fehlerhaft!
          </div>
        </div>
      </div>

    </div>
    <div class="col-md-12 col-lg-2">
      <button type="button" class="btn btn-primary" @click.prevent="switchInOut()">
        Wechseln
        <img src="./assets/arrow-right-arrow-left-solid.svg" width="30" height="30">
      </button>
      <br>
      {{ state.inputBase }} -> {{ state.outputBase }}
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
          <div class="form-floating mb-3">
            <input type="text" class="form-control" id="output" placeholder="Ausgabe" v-model="state.output" readonly
              disabled>
            <label for="output">Ausgabe</label>
          </div>

        </div>
      </div>

    </div>
  </div>
</template>

<style scoped>
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
