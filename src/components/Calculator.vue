<template>
    <div class="calculator">
      <div class="display">
        {{ display }}
      </div>
      <div class="button-grid">
        <operation-input v-for="n in numbers" :operation="n" @clicked="addNumberToStack" />
      </div>
      <div>
        <operation-input v-for="op in allOperations" :operation="op" @clicked="addOperationToStack" />
      </div>
    </div>
</template>

<script lang="ts">
import OperationInput from './OperationInput';

export default {
  name: "Calculator",
  components: {
    OperationInput,
  },
  data() {
    return {
      numbers: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
      mathOperations: ['+', '-', '*', '/'],
      specialOperations: ['=', 'C', 'AC'],
      numberStack: [],
      operationStack: [],
      display: '0'
    }
  },
  computed: {
    allOperations() {
      return this.mathOperations.concat(this.specialOperations)
    }
  },
  methods: {
    addNumberToStack(num){
      this.numberStack.push(num);
      const numericalInput = this.numberStack.join('');
      this.display = numericalInput;
    },
    addOperationToStack(op){
      //Add number to op stack and clear it out
      if (this.numberStack.length !== 0) {
        this.operationStack.push(this.numberStack.join(''));
        this.numberStack = [];
      }

      //Add non-numeric operation to stack, and overwrite last stack entry if is a math op
      if (this.mathOperations.includes(this.operationStack[this.operationStack.length - 1])) {
        this.operationStack.pop()
      }
      this.operationStack.push(op);

      if (this.specialOperations.includes(op)) {
        const output = this.evaluateStack();
      }
    },
    evaluateStack(){
      const lastOp = this.operationStack[this.operationStack.length - 1];

      if (lastOp === 'AC') {
        this.numberStack = [];
        this.operationStack = [];
        this.display = '0';
      } else if (lastOp === 'C') {
        const updatedOperationStack = this.operationStack.slice(0, -2)
        this.display = updatedOperationStack[updatedOperationStack.length - 2] || '0';
        this.operationStack = updatedOperationStack;
      } else {
        let result = [...this.operationStack];
        result.pop()

        while(result.length !== 1) {
          let localResults = [...result];
          let opIndex = result.findIndex(op => op === '*' || op === '/');
          let mathOp = 0;

          if (opIndex !== -1 ) {
            if (result[opIndex] === '*') {
              mathOp = parseInt(result[opIndex - 1]) * parseInt(result[opIndex + 1]));
            } else {
              mathOp = parseInt(result[opIndex - 1]) / parseInt(result[opIndex + 1]));
            }
          } else {
            opIndex = result.findIndex(op => isNaN(op));
            if(result[opIndex] === '+') {
              mathOp = parseInt(result[opIndex - 1]) + parseInt(result[opIndex + 1]));
            } else {
              mathOp = parseInt(result[opIndex - 1]) - parseInt(result[opIndex + 1]));
            }
          }

          result = [...localResults.slice(0, opIndex - 1), mathOp, ...localResults.slice(opIndex + 2)]
        }

        this.display = result[0];
        this.operationStack = result;
        this.numberStack = [];
      }
    },
  }
}
</script>

<style scoped>
.calculator {
    margin: 0 auto;
    height: 500px;
    width: 400px;
    background-color: red;
}

.button-grid {
  display: grid;
  grid-template-columns: repeat(4, 25% [col-end]);
}

.display {
  width: 300px;
  margin: 0 auto;
  height: 25px;
  background-color: darkgray;
  color: blue;
  text-align: right;
}
</style>
