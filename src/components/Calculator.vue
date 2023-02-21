<script lang="ts" setup>
import { onMounted, onUnmounted, ref, computed} from 'vue';

interface CalculatorMemory {
  firstNumber: number,
  secondNumber: number | null,
  operator: string,
  reset: boolean,
}

const Memory: CalculatorMemory = {
  firstNumber: 0,
  secondNumber: null,
  operator: "",
  reset: false,
}

const isDisabled = ref<boolean>(false)

const display = ref<string>("")

const handleNumber = (num: number) => {
  if (Memory.reset) {
    changeDisplay("set", "")
    Memory.reset = false
  }
  if (display.value.length === 1 && display.value[0] === "0") {
    if (num !== 0) {
      changeDisplay("set", num.toString())
    }
  } else {
    changeDisplay("add", num.toString())
  }
}

const handleComma = () => {
  if (Memory.reset) {
    changeDisplay("set", "0,")
    Memory.reset = false
  }
  if (!display.value.includes(",") && display.value) {
    changeDisplay("add", ",")
  }
}

const handleNegative = () => {
  if (display.value) {
    if (display.value.includes("-")) {
      let neagetedNumber: string[] = display.value.split("")
      neagetedNumber.shift()
      changeDisplay("set", neagetedNumber.join(""))
    } else {
      let neagetedNumber: string[] = display.value.split("")
      neagetedNumber.unshift("-")
      changeDisplay("set", neagetedNumber.join(""))
    }
  }
}

const clearMemory = () => {
  display.value = ""
  Memory.firstNumber = 0
  Memory.secondNumber = null
  Memory.operator = ""
  Memory.reset = false
  isRounded.value = false
  if (isDisabled.value) {
    isDisabled.value = false
  }
}

const clearEntry = () => {
  display.value = ""
  if (Memory.reset && Memory.secondNumber) {
    Memory.firstNumber = 0
  }
}

const deleteNumber = () => {
  if (!Memory.reset) {
    let isOneNumberNegated = display.value.length === 2 && display.value[0] === "-"
    if (display.value.length === 1 || isOneNumberNegated) {
      changeDisplay("set", "0")
    } else {
      let displayArr: string[] = display.value.split("")
      displayArr.pop()
      changeDisplay("set", displayArr.join(""))
    }
  }
}

const handleOperator = (sign: string) => {
  // if clicked for the first time
  if (!Memory.operator) {
    setNumber("first", Number(floatNumberFormat("toMemory", display.value)))
    Memory.reset = true
  }

  if (Memory.operator && Memory.secondNumber === null) {
    setNumber("second", Number(floatNumberFormat("toMemory", display.value)))
    handelEqual()
  }
  Memory.secondNumber = null
  Memory.operator = sign
}

const handleSquare = () => {
  if (display.value) {
    let dotValue = floatNumberFormat("toMemory", display.value)
    let calcValue = calculateSafely("square", Number(dotValue))
    let commaValue = floatNumberFormat("toDisplay", calcValue.toString())
    changeDisplay("set", commaValue)
    if (Memory.reset && Memory.secondNumber) {
      setNumber("first", Number(floatNumberFormat("toMemory", display.value)))
    } else {
      setNumber("second", Number(floatNumberFormat("toMemory", display.value)))
    }
    Memory.reset = true
  }
}

const handleSquareRoot = () => {
  if (display.value) {
    if (display.value.includes("-")) {
      throwError()
      return
    }
    let dotValue = floatNumberFormat("toMemory", display.value)
    let calcValue = calculateSafely("root", Number(dotValue))
    let commaValue = floatNumberFormat("toDisplay", calcValue.toString())
    changeDisplay("set", commaValue)
    if (Memory.reset) {
      setNumber("first", Number(floatNumberFormat("toMemory", display.value)))
    } else {
      setNumber("second", Number(floatNumberFormat("toMemory", display.value)))
    }
    Memory.reset = true
  }
}

const handleOneOverX = () => {
  if (display.value) {
    if (display.value === "0") {
      throwError(0)
      return
    }
    let dotValue = floatNumberFormat("toMemory", display.value)
    let calcValue = calculateSafely("overX", Number(dotValue))
    let commaValue = floatNumberFormat("toDisplay", calcValue.toString())
    changeDisplay("set", commaValue)
    if (Memory.reset) {
      setNumber("first", Number(floatNumberFormat("toMemory", display.value)))
    } else {
      setNumber("second", Number(floatNumberFormat("toMemory", display.value)))
    }
    Memory.reset = true
  }
}

const handlePercentage = () => {
  if (display.value) {
    let dotValue = floatNumberFormat("toMemory", display.value)
    let calcValue = calculateSafely("percent", Number(dotValue))
    let commaValue = floatNumberFormat("toDisplay", calcValue.toString())
    changeDisplay("set", commaValue)
    setNumber("second", Number(floatNumberFormat("toMemory", display.value)))
  }
}

const handelEqual = () => {
  if (Memory.reset === false) {
    let dotValue = floatNumberFormat("toMemory", display.value)
    setNumber("second", Number(dotValue))
  }

  if (Memory.secondNumber === null) {
    setNumber("second", Memory.firstNumber)
  }

  if (Memory.operator === "+" && Memory.secondNumber !== null) {
    let calcValue = calculateSafely("add", Memory.firstNumber, Memory.secondNumber)
    let commaValue = floatNumberFormat("toDisplay", calcValue.toString())
    changeDisplay("set", commaValue)
    addToHistory()
    let dotValue = floatNumberFormat("toMemory", display.value)
    setNumber("first", Number(dotValue))
    Memory.reset = true
  }

  if (Memory.operator === "-" && Memory.secondNumber !== null) {
    let calcValue = calculateSafely("sub", Memory.firstNumber, Memory.secondNumber)
    let commaValue = floatNumberFormat("toDisplay", calcValue.toString())
    changeDisplay("set", commaValue)
    addToHistory()
    let dotValue = floatNumberFormat("toMemory", display.value)
    setNumber("first", Number(dotValue))
    Memory.reset = true
  }

  if (Memory.operator === "*" && Memory.secondNumber !== null) {
    let calcValue = calculateSafely("multi", Memory.firstNumber, Memory.secondNumber)
    let commaValue = floatNumberFormat("toDisplay", calcValue.toString())
    changeDisplay("set", commaValue)
    addToHistory()
    let dotValue = floatNumberFormat("toMemory", display.value)
    setNumber("first", Number(dotValue))
    Memory.reset = true
  }

  if (Memory.operator === "/" && Memory.secondNumber !== null) {
    if (Memory.secondNumber === 0) {
      throwError(0)
      return
    }
    let calcValue = calculateSafely("div", Memory.firstNumber, Memory.secondNumber)
    let commaValue = floatNumberFormat("toDisplay", calcValue.toString())
    changeDisplay("set", commaValue)
    addToHistory()
    let dotValue = floatNumberFormat("toMemory", display.value)
    setNumber("first", Number(dotValue))
    Memory.reset = true
  }
}

// --------------------------------- Pomocné funkce ---------------------------------

const throwError = (num?: number) => {
  clearMemory()
  isDisabled.value = true
  if (num === 0) {
    display.value = "Nedělím nulou"
  } else {
    display.value = "Chyba"
  }
}

const isValidNumber = (num: number) => {
  if (
    num <= Number.MAX_SAFE_INTEGER
    &&
    num >= Number.MIN_SAFE_INTEGER
  ) {
    if (num > 0 && num < Number.MIN_VALUE) {
      return false
    }
    return true
  } else {
    return false
  }
}

const setNumber = (position: string, num: number) => {
  if (isValidNumber(num)) {
    if (position === "first") {
      Memory.firstNumber = num
    } else if (position === "second") {
      Memory.secondNumber = num
    }
  } else {
    clearMemory()
    isDisabled.value = true
    display.value = "Velké číslo"
  }
}

const floatNumberFormat = (type: string, value: string) => {
  let valueArray: string[] = value.split("")
  let output: string = ""
  if (type === "toMemory") {
    let formatedValue: string[] = valueArray.map((char) => char === "," ? "." : char)
    output = formatedValue.join("")
  } else if (type === "toDisplay") {
    let formatedValue: string[] = valueArray.map((char) => char === "." ? "," : char)
    output = formatedValue.join("")
  }
  return output
}

const calculateSafely = (method: string, firstNum: number, secondNum: number = 0) => {
  let str1: string = firstNum.toString()
  let str2: string = secondNum.toString()
  let length: number = 1
  let output: number = 0
  if (str1.includes(".") || str2.includes(".")) {
    let arr: number[] = []
    if (str1.includes(".")) {
      arr.push(str1.length - 2)
    }
    if (str2.includes(".")) {
      arr.push(str2.length - 2)
    }
    length = 10 ** Math.max(...arr)
  }
  if (method === "square") {
    output = ((firstNum * length) ** 2) / length ** 2
  }
  if (method === "root") {
    output = Math.sqrt(firstNum)
  }
  if (method === "overX") {
    output = 1 / firstNum
  }
  if (method === "add") {
    output = ((firstNum * length) + (secondNum * length)) / length
  }
  if (method === "sub") {
    output = ((firstNum * length) - (secondNum * length)) / length
  }
  if (method === "multi") {
    output = ((firstNum * length) * (secondNum * length)) / length ** 2
  }
  if (method === "div") {
    output = ((firstNum * length) / (secondNum * length))
  }
  if (method === "percent") {
    output = (Memory.firstNumber / 100) * firstNum
  }
  return output
}

const changeDisplay = (method: string, value: string) => {
  if (method === "set") {
    let valueArray: string[] = value.split("")
    let pureNumber: string[] = valueArray.filter((char) => char !== "," && char !== "-")
    if (pureNumber.length >= 16) {
      let dotArray: string[] = floatNumberFormat("toMemory", valueArray.join("")).split("")
      isRounded.value = true
      display.value = roundNumber(dotArray, 14)
    } else {
      display.value = value
    }
  } else if (method === "add") {
    let valueArray: string[] = (display.value + value).split("")
    let pureNumber: string[] = valueArray.filter((char) => char !== "," && char !== "-")
    if (pureNumber.length < 16) {
      display.value += value
    }
  }
}

const roundNumber = (value: string[], to: number): string => {
  let roundedValue: string[] = Number(Number(value.join("")).toFixed(to)).toString().split("")
  let pureNumber: string[] = roundedValue.filter((char) => char !== "," && char !== "-")
  if (pureNumber.length < 16 || to === 0) {
    let finish: string = floatNumberFormat("toDisplay", roundedValue.join(""))
    return finish
  } else {
    return roundNumber(roundedValue, to - 1)
  }
}

// ------------------------ history system --------------------------

interface Calculation {
  first: number,
  second: number,
  sign: string,
  outcome: number
}

const history = ref<Calculation[]>([])

const addToHistory = () => {
  if (Memory.secondNumber === null) return
  const newCalculation: Calculation = {
    first: Memory.firstNumber,
    second: Memory.secondNumber,
    sign: Memory.operator,
    outcome: Number(floatNumberFormat("toMemory", display.value))
  }
  history.value.unshift(newCalculation);
}

const deleteHistory = () => {
  history.value = []
}


// --------------------- Lifecycles ---------------------

const numpad: string[] = ["1", "2", "3", "4", "5", "6", "7", "8", "9", "0"]
const signs: string[] = ["+", "-", "*", "/"]

type KeyboardAction = {
  match: (e: KeyboardEvent) => boolean,
  action: (e: KeyboardEvent) => void
}


const keyboardActions: KeyboardAction[] = [
  {
    match: (e) => numpad.includes(e.key),
    action: (e) => handleNumber(Number(e.key))
  },
  {
    match: (e) => signs.includes(e.key),
    action: (e) => handleOperator(e.key)
  },
  {
    match: (e) => e.key === ",",
    action: (e) => handleComma()
  },
  {
    match: (e) => e.key === "Enter",
    action: (e) => handelEqual()
  },
  {
    match: (e) => e.key === "Backspace",
    action: (e) => deleteNumber()
  },
  {
    match: (e) => e.key === "Escape",
    action: (e) => clearMemory()
  },
  {
    match: (e) => e.key === "Delete",
    action: (e) => clearEntry()
  },
]

const handleKeyboard = (e: KeyboardEvent) => {
  const keyboardAction = keyboardActions.find((keyboardAction) => keyboardAction.match(e))
  if (!keyboardAction) return
  e.preventDefault()
  keyboardAction.action(e)
}

onMounted(() => {
  window.addEventListener("keyup", handleKeyboard)
})

onUnmounted(() => {
  window.removeEventListener("keyup", handleKeyboard)
})

// --------------------- CSS animation ----------------------
const handleSidebar = () => {
  if (sidebar.value.isShown) {
    sidebar.value.translate = 4
  } else {
    sidebar.value.translate = 504
  }
  sidebar.value.isShown = !sidebar.value.isShown
}

const sidebar = ref({
  isShown: false,
  translate: 4
})

const displayFontSize = computed(() => {
  if (display.value) {
    return display.value.length > 10 ? (10 / display.value.length) + 3.1 : 6
  }
})

const isRounded = ref<boolean>(false)

</script>

<template>
  <div class="container">
    <div class="sidePanel" :style="{ translate: `${sidebar.translate}px` }">
      <span>Historie</span>
      <button class="clearHistory" @click="deleteHistory">&#10006;</button>
      <template v-for="calculations in history">
        <p>{{ calculations.first }} {{ calculations.sign }} {{ calculations.second }} =</p>
        <p>{{ calculations.outcome }}</p>
        <hr>
      </template>
    </div>
    <div class="frontContainer">
      <div class="diode"
        :data-tooltip="isRounded ? 'Čísla jsou nepřesná. Během výpočtu došlo k zaokrouhlení' : 'Čísla jsou přesná'" :style="{
          backgroundColor: isRounded ? '#d60000cc' : '#bdbdbdcc',
          boxShadow: isRounded ? '0 0 1rem .8rem #d600004d' : '0 0 1rem .8rem #bdbdbd4d'
        }">
      </div>
      <button class="sidebarButton" @click="handleSidebar">{{ sidebar.isShown ? "<<" : ">>" }}</button>
      <div class="display" :style="{ fontSize: displayFontSize + 'rem' }">{{ display }}</div>
      <div class="buttons">
        <button @click="handlePercentage" :disabled="isDisabled">%</button>
        <button @click="clearEntry" :disabled="isDisabled">CE</button>
        <button @click="clearMemory">C</button>
        <button @click="deleteNumber" :disabled="isDisabled">del</button>
        <button @click="handleOneOverX" :disabled="isDisabled">&#8543;<sub>x</sub></button>
        <button @click="handleSquare" :disabled="isDisabled">x<sup>2</sup></button>
        <button @click="handleSquareRoot" :disabled="isDisabled">&radic;x</button>
        <button @click="$event => handleOperator('/')" :disabled="isDisabled">&divide;</button>
        <button @click="$event => handleNumber(7)" :disabled="isDisabled">{{ 7 }}</button>
        <button @click="$event => handleNumber(8)" :disabled="isDisabled">{{ 8 }}</button>
        <button @click="$event => handleNumber(9)" :disabled="isDisabled">{{ 9 }}</button>
        <button @click="$event => handleOperator('*')" :disabled="isDisabled">&times;</button>
        <button @click="$event => handleNumber(4)" :disabled="isDisabled">{{ 4 }}</button>
        <button @click="$event => handleNumber(5)" :disabled="isDisabled">{{ 5 }}</button>
        <button @click="$event => handleNumber(6)" :disabled="isDisabled">{{ 6 }}</button>
        <button @click="$event => handleOperator('-')" :disabled="isDisabled">-</button>
        <button @click="$event => handleNumber(1)" :disabled="isDisabled">{{ 1 }}</button>
        <button @click="$event => handleNumber(2)" :disabled="isDisabled">{{ 2 }}</button>
        <button @click="$event => handleNumber(3)" :disabled="isDisabled">{{ 3 }}</button>
        <button @click="$event => handleOperator('+')" :disabled="isDisabled">+</button>
        <button @click="handleNegative" :disabled="isDisabled">&plusmn;</button>
        <button @click="$event => handleNumber(0)" :disabled="isDisabled">{{ 0 }}</button>
        <button @click="handleComma" :disabled="isDisabled">,</button>
        <button @click="handelEqual" :disabled="isDisabled">=</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  align-items: center;
}

.frontContainer {
  position: relative;
  width: 500px;
  height: 700px;
  padding: .4rem;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: 0.3fr 1.5fr repeat(6, 1fr);
  grid-template-areas:
    "diode . . history"
    "display display display display"
    "button button button button"
    "button button button button"
    "button button button button"
    "button button button button"
    "button button button button"
    "button button button button";
  gap: .2rem;
  box-shadow: 0px 0px .4rem .4rem rgba(0, 0, 0, 0.2);
  border-radius: 8px;
  background-color: #616161;
}

.display {
  box-sizing: border-box;
  grid-area: display;
  background-color: #212121;
  margin-bottom: .3rem;
  margin-inline: .3rem;
  padding: 0.4rem;
  text-align: right;
}

.buttons {
  display: grid;
  grid-area: button;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(6, 1fr);
  gap: .1rem;
}

.sidebarButton {
  grid-area: history;
  justify-self: end;
  min-width: 2rem;
  min-height: 2rem;
  font-size: 1.5rem;
  text-align: center;
  border-radius: 1rem;
}

.sidePanel {
  position: absolute;
  max-width: 500px;
  height: 700px;
  background-color: #212121;
  border: .3rem solid #616161;
  border-radius: 10px;
  border-left: none;
  text-align: right;
  font-size: 2rem;
  padding: .8rem;
  box-sizing: border-box;
  box-shadow: .1rem 0px .4rem .3rem rgba(0, 0, 0, 0.2);
  transition: all 0.6s ease;
  overflow-y: scroll;
}

.sidePanel>p {
  margin-block: .3rem;
  font-size: 1.5rem;
}

.sidePanel>hr {
  border-color: #26A69A;
}

.sidePanel>span {
  font-size: 2.3rem;
  font-weight: bold;
}

.clearHistory {
  background: none;
  margin-inline: .2rem;
  color: #B71C1C;
}

.clearHistory:hover {
  color: hsla(0, 73%, 41%, 0.5);
}

.clearHistory:active {
  color: hsla(0, 73%, 41%, 0.8);
}

.diode {
  grid-area: diode;
  width: 1.5rem;
  height: 1.5rem;
  border-radius: .75rem;
  position: relative;
}

.diode:hover::before {
  position: absolute;
  content: attr(data-tooltip);
  width: max-content;
  padding: .5rem;
  font-size: 1.2rem;
  background-color: #212121bd;
  transform: translateX(1.5rem) translateY(1rem);
}

::-webkit-scrollbar {
  width: .7rem;
}
::-webkit-scrollbar-track {
  background-color: #212121;
  border-radius: 100vw;
  margin-block: .5rem;
}
::-webkit-scrollbar-thumb {
  background-color: #26A69A;
  border-radius: 100vw;
  border: .35rem solid #212121;
}

::-webkit-scrollbar-thumb:hover {
  border: .05rem solid #212121;
}

</style>
