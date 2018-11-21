<template>
  <v-layout column justify-center align-center>
    <v-flex xs12 sm8 md6>
      <v-card>
        <v-card-title class="headline">Calculate App</v-card-title>
        <v-container fluid>
          <v-text-field
            v-model="inputValue"
            color="cyan darken"
            placeholder="I'm ready."
            @input="inputChange"
          />
          = {{ answer }}
        </v-container>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
const MATH = {
  '+': { priority: 2, f: (x, y) => Number(x) + Number(y) },
  '-': { priority: 2, f: (x, y) => Number(x) - Number(y) },
  '*': { priority: 3, f: (x, y) => Number(x) * Number(y) },
  '/': { priority: 3, f: (x, y) => Number(x) / Number(y) }
}
const OPERATORS = Object.keys(MATH).join('')
const NUMBERS = '0123456789'
const LEFT_BRACKET = '('
const RIGHT_BRACKET = ')'
const WHITE_SPACE = ' '
const DOT = '.'

export default {
  components: {},
  data: () => ({
    inputValue: '',
    answer: '',
    style: {}
  }),
  methods: {
    inputChange(inputText) {
      inputText = inputText
        .replace(/ /g, '')
        .replace(/(\d+\.)([^\d])/, '$10$2')
        .replace(/(\d+\.)$/, '$10')
        .replace(/(^[-|+])\(/, '$11*(')
        .replace(/^([-|+]\d+)/, '(0$1)')
        .replace(/([^\d|^)])([-|+]\d+)/g, '$1(0$2)')
        .replace(/([^\d])\./g, '$10.')
        .replace(/^(\.\d+)/, '0$1')
      console.log(inputText, 777)
      if (this.inValidCharCheck(inputText) === true && inputText !== '') {
        const rpnList = this.rpn(inputText)
        console.log('a:', rpnList)
        this.answer = String(this.calculateByRPN(rpnList))
      } else if (inputText == '') {
        this.answer = ''
      } else {
        this.answer = 'well...'
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
        return true
      } else {
        const includeInValidCharErr =
          'invalid-character: [' +
          wordsList[falseIndex] +
          '], l: [' +
          (falseIndex + 1) +
          ']'
        return includeInValidCharErr
      }
    },
    rpn(inputText) {
      let stack = []
      let rpnList = []
      let normalFormulaList = []

      while (1) {
        let matchValue = inputText.match(/\d+(?:\.\d+)?/)

        if (matchValue !== null) {
          normalFormulaList.push(matchValue[0])
          inputText = inputText.replace(matchValue[0], '')
        } else {
          normalFormulaList.push(inputText[0])
          inputText = inputText.slice(1)
        }

        if (inputText === '') {
          break
        }
      }

      // console.log(normalFormulaList, 111)

      for (const token of normalFormulaList) {
        console.log('AA', token, rpnList)
        if (token === '.') {
          return []
        }
        if (token.match(/\d+(?:\.\d+)?/) !== null) {
          rpnList.push(token)
          continue
        } else if (token === RIGHT_BRACKET) {
          while (1) {
            if (stack.length === 0) {
              break
            }
            console.log(stack)
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
                if (MATH[token]['priority'] > MATH[topStack]['priority']) {
                  // rpnList.push(stack.pop())
                  stack.push(token)
                  break
                } else {
                  // stack.push(token)
                  rpnList.push(stack.pop())
                  // break
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
        rpnList.push(stack.pop())
      }
      console.log('result: ', rpnList)
      return rpnList
    },
    calculateByRPN(rpnList) {
      let stack = []
      let accumulator = []

      for (const token of rpnList) {
        const existNumber = token.match(/\d+(?:\.\d+)?/) !== null
        const existOperator = OPERATORS.indexOf(token) > -1
        if (existNumber || existOperator) {
          if (existNumber) {
            stack.push(token)
          } else if (existOperator) {
            let accumulator = stack.pop()
            accumulator = MATH[token]['f'](stack.pop(), accumulator)
            if (isNaN(accumulator) === true) {
              return 'well...'
            }

            stack.push(accumulator)
          }
        } else {
          if (token !== LEFT_BRACKET && token !== RIGHT_BRACKET) {
            console.log('undefined token: ' + token)
            return 'well...'
          }
        }
      }

      if (stack.length !== 1) {
        console.log('failed calculate.')
        console.log('now stack: ' + stack)
        return 'well...'
      }

      return stack[0]
    }
  }
}
</script>
