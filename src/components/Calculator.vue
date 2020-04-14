<template>
    <div class="calculator">
      <div class="body">
        <div class="display">
          {{ display }}
        </div>
        <div class="number-grid">
          <operation-input v-for="n in numbers" :operation="n" @clicked="addNumberToStack" />
        </div>
        <div class="operation-grid">
          <operation-input v-for="op in allOperations" :operation="op" @clicked="addOperationToStack" />
        </div>
      </div>
    </div>
</template>

<script>
import OperationInput from './OperationInput';

export default {
  name: "Calculator",
  components: {
    OperationInput,
  },
  data() {
    return {
      numbers: ['7', '8', '9', '4', '5', '6', '1', '2', '3', '0', ],
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
              mathOp = parseInt(result[opIndex - 1]) * parseInt(result[opIndex + 1]);
            } else {
              mathOp = parseInt(result[opIndex - 1]) / parseInt(result[opIndex + 1]);
            }
          } else {
            opIndex = result.findIndex(op => isNaN(op));
            if(result[opIndex] === '+') {
              mathOp = parseInt(result[opIndex - 1]) + parseInt(result[opIndex + 1]);
            } else {
              mathOp = parseInt(result[opIndex - 1]) - parseInt(result[opIndex + 1]);
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
    /* overall display */
    margin: 0 auto;
    background-color: rgb(0, 0, 0, 0.8);
    border-radius: 5px;
    font-family: 'Roboto', sans-serif;

    /* possibly remove dimensions */
    height: 300px;
    width: 350px;
}

.body {
  /* layout of contents */
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 100%;
  justify-content: space-between;
}

.number-grid {
  display: grid;
  grid-template-columns: repeat(3, 98px);
  grid-template-rows: repeat(4, 45px);
  grid-gap: 3px;
}

.operation-grid {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  width: 300px;
  margin-bottom: 10px;
}

.display {
  width: 300px;
  background-color: darkolivegreen;
  padding: 3px 3px 3px 0px;
  color: black;
  text-align: right;
  margin: 10px auto 0px auto;
  border-radius: 3px;
}
</style>
