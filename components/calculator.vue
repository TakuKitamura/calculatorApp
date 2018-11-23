<template>
    <div class="calculator-border">
        <div class="calculator-display">
            <v-text-field
                v-cloak
                v-model="formula"
                class="calculator-formula"
                color="cyan darken"
                placeholder="I'm ready."
                autofocus
                @input="inputChange"
                @keyup.escape="cleanAll()"
            />
            <div v-cloak class="calculator-result">= {{ result }}</div>
        </div>
        <div class="calculator-items">
            <div class="calculator-row-1">
                <div class="calculator-item" @click="drop()">DEL</div>
                <div class="calculator-item" @click="cleanAll()">C</div>
                <div class="calculator-item" @click="toggle()">±</div>
                <div class="calculator-item" @click="addMark('(')">(</div>
                <div class="calculator-item" @click="addMark(')')">)</div>
            </div>
            <div class="calculator-row-2">
                <div class="calculator-item" @click="addMark('7')">7</div>
                <div class="calculator-item" @click="addMark('8')">8</div>
                <div class="calculator-item" @click="addMark('9')">9</div>
                <div class="calculator-item" @click="addMark('/')">/</div>
                <div class="calculator-item" @click="addMark('%')">%</div>
            </div>
            <div class="calculator-row-3">
                <div class="calculator-item" @click="addMark('4')">4</div>
                <div class="calculator-item" @click="addMark('5')">5</div>
                <div class="calculator-item" @click="addMark('6')">6</div>
                <div class="calculator-item" @click="addMark('*')">*</div>
                <div class="calculator-item" @click="addMark('^')">^</div>
            </div>
            <div class="calculator-row-4">
                <div class="calculator-item" @click="addMark('1')">1</div>
                <div class="calculator-item" @click="addMark('2')">2</div>
                <div class="calculator-item" @click="addMark('3')">3</div>
                <div class="calculator-item" @click="addMark('-')">-</div>
                <div class="calculator-item" @click="addMark('+')">+</div>
            </div>
            <div class="calculator-row-5">
                <div class="calculator-item" @click="addMark('0')">0</div>
                <div class="calculator-item" @click="addMark('00')">00</div>
                <div class="calculator-item" @click="addMark('.')">.</div>
            </div>
        </div>
    </div>
</template>

<script>
const MATH = {
  '^': { priority: 2, f: (x, y) => Number(x) ** Number(y) },
  '*': { priority: 3, f: (x, y) => Number(x) * Number(y) },
  '/': { priority: 3, f: (x, y) => Number(x) / Number(y) },
  '%': { priority: 3, f: (x, y) => Number(x) % Number(y) },
  '+': { priority: 4, f: (x, y) => Number(x) + Number(y) },
  '-': { priority: 4, f: (x, y) => Number(x) - Number(y) }
}
const OPERATORS = Object.keys(MATH).join('')
const NUMBERS = '0123456789'
const LEFT_BRACKET = '('
const RIGHT_BRACKET = ')'
const WHITE_SPACE = ' '
const DOT = '.'
export default {
  name: 'CalculatorContent',
  data: () => ({
    formula: '',
    result: ''
  }),
  methods: {
    drop() {
      this.formula = this.formula.slice(0, -1)
      this.inputChange(this.formula)
    },
    cleanAll() {
      this.formula = ''
      this.inputChange(this.formula)
    },
    toggle() {
      if (this.result !== '' && isNaN(Number(this.result)) === false) {
        this.formula = String(Number(this.result) * -1)
        this.inputChange(this.formula)
      }
    },
    addMark(v) {
      this.formula += v
      this.inputChange(this.formula)
    },
    inputChange(inputText) {
      inputText = inputText
        .replace(/ /g, '')

        .replace(/(\d+\.)([^\d])/g, '$10$2') // 3.* -> 3.0*
        .replace(/(\d+\.)$/, '$10') // 3.$ -> 3.0$
        .replace(/([^\d])\./g, '$10.') // *. -> *0.
        .replace(/^(\.\d+)/, '0$1') //^.2 ->^ 0.2

        .replace(/([-|+])\(/g, '$11*(') // -( -> -1*(
        .replace(/^([-|+]\d+\.?\d*|\.\d+)/, '(0$1)') // ^-3.0 -> ^0-3.0
        .replace(/([^\d|^)])([-|+]\d+\.?\d*|\.\d+)/g, '$1(0$2)') // *-2.0 -> *(0-2.0)

      const err = this.inValidCharCheck(inputText)

      if (err !== null) {
        this.result = this.escapeHtml(String(err))
        return
      }

      if (inputText !== '') {
        const [rpnList, err] = this.rpn(inputText)
        if (err !== null) {
          this.result = this.escapeHtml(err)
          return
        }
        const result = this.calculateByRPN(rpnList)
        this.result = this.escapeHtml(String(this.calculateByRPN(rpnList)))
      } else if (inputText == '') {
        this.result = ''
      }
    },
    inValidCharCheck(inputText) {
      const wordsList = inputText.split('')
      const areValid = wordsList.map(vv => {
        const validMark =
          NUMBERS + OPERATORS + LEFT_BRACKET + RIGHT_BRACKET + DOT + WHITE_SPACE

        return validMark.indexOf(vv) > -1
      })

      const falseIndex = areValid.indexOf(false)

      if (falseIndex === -1) {
        const err = null
        return err
      } else {
        const err =
          'unexpected: [' +
          wordsList[falseIndex] +
          '], l: [' +
          (falseIndex + 1) +
          ']'
        return err
      }
    },
    rpn(inputText) {
      let stack = []
      let rpnList = []
      let normalFormulaList = []

      while (1) {
        let matchNumber = inputText.match(/\d+\.?\d*|\.\d+/)
        let matchOperator = inputText.match(/[^\d]/)

        if (matchNumber !== null) {
          if (matchNumber.index === 0) {
            normalFormulaList.push(matchNumber[0])
            inputText = inputText.replace(matchNumber[0], '')
            continue
          }
        }

        if (matchOperator !== null) {
          if (matchOperator.index === 0) {
            normalFormulaList.push(matchOperator[0])
            inputText = inputText.slice(1)
            continue
          }
        }

        if (inputText === '') {
          break
        }
      }
      console.log(normalFormulaList)

      for (const token of normalFormulaList) {
        if (token === '.') {
          const err = 'unexpected [.].'
          return [null, err]
        }
        if (token.match(/\d+(?:\.\d+)?/) !== null) {
          rpnList.push(token)
          continue
        } else if (token === RIGHT_BRACKET) {
          while (1) {
            if (stack.length === 0) {
              const err = 'mismatch bracket.'
              return [null, err]
            }

            const topStack = stack[stack.length - 1]
            if (topStack === LEFT_BRACKET) {
              stack.pop()
              break
            } else {
              rpnList.push(stack.pop())
            }
          }
          continue
        } else if (token === LEFT_BRACKET) {
          stack.push(token)
          continue
        } else {
          while (1) {
            if (stack.length === 0) {
              stack.push(token)
              break
            } else {
              const topStack = stack[stack.length - 1]
              if (topStack !== LEFT_BRACKET) {
                if (MATH[token] === undefined) {
                  const err = 'undefined symbol.'
                  return [null, err]
                }
                if (MATH[token]['priority'] < MATH[topStack]['priority']) {
                  // rpnList.push(stack.pop())
                  stack.push(token)
                  break
                } else {
                  rpnList.push(stack.pop())
                }
              } else {
                stack.push(token)
                break
              }
            }
          }
        }
      }

      const stackLength = stack.length
      for (let i = 0; i < stackLength; i++) {
        if (stack[i] === LEFT_BRACKET || stack[i] === RIGHT_BRACKET) {
          const err = 'mismatch bracket.'
          return [null, err]
        }
        rpnList.push(stack.pop())
      }

      if (rpnList.includes(LEFT_BRACKET) || rpnList.includes(RIGHT_BRACKET)) {
        const err = 'mismatch bracket.'
        return [null, err]
      }

      if (rpnList.length === 0) {
        const err = 'unknown error.'
        return [null, err]
      }

      console.log('result: ', rpnList)
      const err = null
      return [rpnList, err]
    },
    calculateByRPN(rpnList) {
      let stack = []
      let accumulator = []

      for (const token of rpnList) {
        const existNumber = token.match(/\d+\.?\d*|\.\d+/) !== null
        const existOperator = OPERATORS.indexOf(token) > -1
        if (existNumber || existOperator) {
          if (existNumber) {
            stack.push(token)
          } else if (existOperator) {
            let accumulator = stack.pop()
            if (MATH[token] === undefined) {
              const err = 'unexpected token: [' + token + '].'
              console.log(err)
              return NaN
            }
            accumulator = MATH[token]['f'](stack.pop(), accumulator)
            if (isNaN(accumulator) === true) {
              const err = 'unexpected accumulator: [' + accumulator + '].'
              console.log(err)
              return this.formula
            }

            stack.push(accumulator)
          }
        } else {
          if (token !== LEFT_BRACKET && token !== RIGHT_BRACKET) {
            console.log('undefined token: ' + token)
            return NaN
          }
        }
      }

      if (stack.length > 1) {
        if (stack[1] === '.') {
          const err = 'unexpected [.].'
          return err
        }
        console.log('failed calculate.')
        console.log('now stack: ' + stack)
        return NaN
      }

      return stack[0]
    },
    escapeHtml(unsafe) {
      return unsafe
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#039;')
    }
  }
}
</script>

<style>
/*vue*/
[v-cloak] {
  display: none;
}

.calculator .calculator-border .calculator-display {
  border: 0.15rem solid #b9b4b4;
  background-color: #37474f;
  margin: 1rem;
  /*width: 24rem;*/
  height: 7rem;
  width: 92.3%;
  font-size: 2rem;
  /*height:18.4%;*/
  -webkit-border-radius: 0.4rem;
  -moz-border-radius: 0.4rem;
  -ms-border-radius: 0.4rem;
  -o-border-radius: 0.4rem;
  border-radius: 0.4rem;
  -webkit-box-shadow: 1rem 1rem 0.5rem #263238;
  box-shadow: 0.1rem 0.1rem 0.5rem #263238;
}

.calculator .calculator-border .calculator-formula {
  /*border: 1px solid red;*/
  width: 100%;
  height: 42.9%;
  text-align: left;
  box-sizing: border-box;
  padding: 0.5rem 1rem;
  color: #ce3b3b;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}

.calculator .calculator-border .calculator-result {
  /*border: 1px solid blue;*/
  width: 100%;
  height: 57.1%;
  text-align: left;
  box-sizing: border-box;
  padding: 0.5rem 1rem;
  color: white;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}

.calculator .calculator-border .calculator-items {
  /*border: 1px solid black;*/
  margin: 3rem 1rem;
  text-align: left;
  width: 92.3%;
  height: auto;
}

.calculator .calculator-border .calculator-items .calculator-row-1,
.calculator .calculator-border .calculator-items .calculator-row-2,
.calculator .calculator-border .calculator-items .calculator-row-3,
.calculator .calculator-border .calculator-items .calculator-row-4,
.calculator .calculator-border .calculator-items .calculator-row-5 {
  /*border: 1px solid blue;*/
  margin-bottom: 2.3%;
  width: 100%;
}

.calculator .calculator-border .calculator-items .calculator-item {
  border: 0.1rem solid #455a64;
  width: 16.666%;
  height: 4rem;
  display: inline-block;
  text-align: center;
  line-height: 4rem;
  margin-right: 2.1%;
  color: white;
  cursor: pointer;
  -webkit-border-radius: 0.5rem;
  -moz-border-radius: 0.5rem;
  -ms-border-radius: 0.5rem;
  -o-border-radius: 0.5rem;
  border-radius: 0.5rem;
  /* background-color: rgba(244, 135, 135, 0.78); */
  -webkit-box-shadow: 0.1rem 0.1rem 0.3rem #37474f;
  box-shadow: 0.1rem 0.1rem 0.3rem #37474f;
  transition: all 0.1s;
  -webkit-transition: all 0.1s;
}

.calculator .calculator-border .calculator-items .calculator-item:hover {
  color: #f1f5f7;
  -webkit-box-shadow: 0.3rem 0.3rem 0.5rem #263238;
  box-shadow: 0.3rem 0.3rem 0.5rem #263238;
}

.calculator .calculator-border .calculator-items .calculator-item:active {
  transform: translate(0.05rem, 0.05rem);
  -moz-transform: translate(0.05rem, 0.05rem);
  -webkit-transform: translate(0.05rem, 0.05rem);
}

.calculator
  .calculator-border
  .calculator-row-5
  .calculator-item:nth-child(odd) {
  width: 37.2%;
}

.calculator .calculator-border .calculator-row-1 .calculator-item:nth-child(5),
.calculator .calculator-border .calculator-row-2 .calculator-item:nth-child(5),
.calculator .calculator-border .calculator-row-3 .calculator-item:nth-child(5),
.calculator .calculator-border .calculator-row-4 .calculator-item:nth-child(5),
.calculator .calculator-border .calculator-row-5 .calculator-item:nth-child(3) {
  margin-right: 0;
}
</style>