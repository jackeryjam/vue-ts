<template>
  <b-container :fluid="boardScale >= 10">
    <b-row>
      <b-jumbotron class="text-center">
        <h1 class="display-3">{{title}}</h1>
        <p class="lead">{{tip}}</p>
        <hr class="my-4">
        <p>It uses utility classes for typography and spacing to space content out within the larger container.</p>
      </b-jumbotron>
    </b-row>
    <b-row class="d-flex pb-4">
      <b-card-group deck class="flex-1 mr-2">
        <b-card mw-40>
          <h2 slot="header" class="mb-0">dashboard</h2>
          <b-form>
            <b-form-group :label="'Scale of the chess board is <b>' + boardScale + '</b>'" label-for="chess-board-scale"
                          description="The scale greater then will not show in UI.">
              <b-form-input type="range" min="2" max="12" step="1" 
                            class="form-control" id="chess-board-scale" 
                            v-model="boardScale"
              ></b-form-input>
            </b-form-group>
            <h5 class="mt-3">Choose mode</h5>
            <b-form-radio-group id="modeRadio" v-model="mode" name="modeRadio">
              <b-form-radio value="checkMode">Check mode</b-form-radio>
              <b-form-radio value="runMode">Run mode</b-form-radio>
            </b-form-radio-group>
            <span :class="{ 'd-none': mode != 'checkMode'}">
              <h5 class="mt-3">Please click the chess board on the right, then click check button to verify whether your input is legal!</h5>
              <b-button variant="primary" @click="check">check</b-button>
            </span>
            <span :class="{ 'd-none': mode == 'checkMode'}">
              <h5 class="mt-3">Please choose the run mode</h5>
              <b-form-radio-group id="runMode" v-model="runMode" name="runMode">
                <b-form-radio value="stepByStep">step by step</b-form-radio>
                <b-form-radio value="getResultDirectly">get result directly</b-form-radio>
              </b-form-radio-group>
              <span :class="{ 'd-none': runMode != 'stepByStep'}">
                <h5 class="mt-3">In this mode, you can press next step to continue, press stop to break down, press current state to the lastest step in sever!</h5>
                <b-button-group>
                  <b-button variant="primary" @click="changeStepRunState('previous step')">previous step</b-button>
                  <b-button variant="primary" @click="changeStepRunState('next step')">next step</b-button>
                  <b-button :variant="stepRunState == 'auto run' ? '' :'primary'" @click="changeStepRunState('auto run')">auto run</b-button>
                  <b-button variant="primary" @click="changeStepRunState('restart')">restart</b-button>
                </b-button-group>
                <b-form-group :label="'Scale of the chess board is <b>' + runSpeed + '</b>'" label-for="run-speed"
                              description="The scale greater then will not show in UI.">
                  <b-form-input type="range" min="1" max="8" step="1" 
                                class="form-control" id="run-speed" 
                                v-model="runSpeed"
                  ></b-form-input>
                </b-form-group>
              </span>
              <span :class="{ 'd-none': runMode != 'getResultDirectly'}">
                <h5 class="mt-3">In this mode, you can directly get the result and see the successful answers in the follow as the scale you like.The total solutions is {{solution.solutionNum}}.This is the No.{{directResult + 1}} result</h5>
                </b-form-group> 
                <b-button variant="primary" @click="changeDirectRes(-1)">Previous result</b-button>
                <b-button variant="primary" @click="changeDirectRes(1)">next result</b-button>
                <br>
              </span>
            </span>
          </b-form>
        </b-card>
      </b-card-group>
      <b-card-group>
        <b-card no-body> 
          <b-card-body class="d-flex flex-column justify-content-center">
            <table border="1">
              <tr v-for="(row,i) in chessBoard" v-if="i <= boardScale" :key="i">
                <td v-for="(cell,j) in row" v-if="j <= boardScale" :key="j" :class="{ cell: (i+j)%2 == 0 }" @click="mode == 'checkMode' ? changeQueen(i,j) : ''">
                  <img v-if="cell" class="queen" src="../assets/queen.png"/>
                </td>
              </tr>
            </table>
          </b-card-body>
        </b-card>
      </b-card-group>
    </b-row>
    <b-modal v-model="modal.show" :title="modal.title" @ok="modal.handleOk != undefined ? modal.handleOk : ''">
      <p class="my-4">{{modal.content}}</p>
    </b-modal>
  </b-container>
</template>

<script lang="ts">
  import { Component, Vue, Watch } from 'vue-property-decorator'
  import { } from 'bootstrap-vue/lib/components'
  import $ from 'jquery'
  import { API } from '../service'
  
  interface handleFun{
    ():void
  }
  interface Modal {
      show: boolean;
      title?: string;
      content?: string;
      handleOk?: handleFun;
  }

  interface Solution {
    left: Array < Array< Array<boolean> > >;
    middle: Array < Array< Array<boolean> > >;
  }
  interface SolutionRes {
      size?: number;
      solutionNum?: number;
      solutions?: Solution;
  }

  @Component({
  })
  export default class EightQueen extends Vue {
    title: string = 'Hello,queens!'
    tip: string = 'input the number of queen in the left'
    len: number = 8
    boardScale: number = 6
    chessBoard: Array< Array<boolean> > = []
    mode: string = 'checkMode'
    runMode: string = 'stepByStep'
    email: string = ''
    directResult: number = 0
    runSpeed: number = 1
    stepRunState: string = ''
    modalShow: Boolean = false
    modal:Modal = { show: false }
    solution: SolutionRes = {}
    mounted () {
      this.chessBoard = this.creatBoard(this.boardScale)
    }

    @Watch('boardScale')
    boardScaleChange (val) {
      this.chessBoard = this.creatBoard(parseInt(val))
      if (this.runMode === 'getResultDirectly') {
        this.getResult()
      }
    }

    @Watch('directResult')
    directResultChange (val) {
      if (this.solution === null) return
      this.chessBoard = this.getSolution(this.directResult)
    }

    changeDirectRes (val: string) {
      let num = parseInt(val)
      this.directResult = (num + this.directResult) % this.solution.solutionNum
    }

    creatBoard (scale: number): Array< Array<boolean> > {
      var board: Array< Array<boolean> > = []
      for (let i = 0; i < scale; i++) {
        board[i] = []
        for (let j = 0; j < scale; j++) {
          board[i][j] = false
        }
      }
      return board
    }

    changeQueen (x, y) {
      this.chessBoard[x].splice(y, 1, !this.chessBoard[x][y])
    }

    changeStepRunState (state: string) {
      this.stepRunState = state
    }

    @Watch('runMode')
    runModeChange (val) {
      if (val === 'auto run') {
      } else if (val === 'getResultDirectly') {
        this.getResult()
      }
    }

    check () {
      let payload = {
        'chessBoard': JSON.stringify(this.chessBoard)
      }
      $.post(API.check, payload).then((data, state) => {
        if (state !== 'success') return
        this.modal = {
          show: true,
          title: 'result',
          content: data.isLegal ? 'Congradulation! Your input is legal.' : 'Illegal!!!'
        }
      })
    }

    getResult () {
      this.directResult = 0
      let payload = {
        'size': this.boardScale
      }
      $.get(API.solution, payload).then((data, state) => {
        this.solution = data
        this.chessBoard = this.solution.solutions.left[this.directResult]
      })
    }

    getSolution (index: number):Array< Array<boolean> > {
      let left = this.solution.solutions.left.length
      let middle = this.solution.solutions.middle.length
      if (index < left) {
        return this.solution.solutions.left[index]
      } else if (index < left + middle) {
        return this.solution.solutions.middle[index - left]
      } else {
        return this.solution.solutions.left[index - left - middle].map(val => [...val].reverse())
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
$cell-side : 100px;
$queen-side : $cell-side * 0.9;
table td{
  width: $cell-side;
  height: $cell-side;
  text-align: center;
}
.cell{
  background: black;
}
.queen{
  width: $queen-side;
  height: $queen-side;
}
.jumbotron{
  width: 100%;
}
@for $i from 1 to 10 {
  .flex-#{$i}{
    flex: $i;
  }
}
</style>
