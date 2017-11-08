<template>
  <b-container>
    <b-row>
      <b-jumbotron class="text-center">
        <h1 class="display-3">{{title}}</h1>
        <p class="lead">{{tip}}</p>
        <hr class="my-4">
        <p>It uses utility classes for typography and spacing to space content out within the larger container.</p>
      </b-jumbotron>
    </b-row>
    <b-row class="d-flex pb-4">
      <b-card-group deck class="flex-1" :class="{ 'mr-2': !(mode == 'runMode' && runMode == 'getResultDirectly')}">
        <b-card>
          <h2 slot="header" class="mb-0">dashboard</h2>
          <b-form>
            <b-form-group :label="'Scale of the chess board is <b>' + boardScale + '</b>'" label-for="chess-board-scale"
                          description="The scale greater then will not show in UI.">
              <b-form-input type="range" min="2" max="8" step="1" 
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
                <h5 class="mt-3">In this mode, you can directly get the result and see the successful answers in the follow.Or you can input the greater scale and your email, once the result come out,you can receive a email.</h5>
                <b-form-group :label="'Scale of the run directly is  <b>' + directRunScale + '</b>'" label-for="direct-run-scale"
                              description="The scale greater then will not show in UI.">
                  <b-form-input type="range" min="2" max="16" step="1" 
                                class="form-control" id="direct-run-scale" 
                                v-model="directRunScale"
                  ></b-form-input>
                </b-form-group> 
                <b-form-group id="emailInputtGroup"
                              label="Email address:" label-for="emailInput"
                              description="We'll never share your email with anyone else.">
                  <b-form-input id="emailInput"
                                type="email" v-model="email" required
                                placeholder="Enter email"
                  ></b-form-input>
                </b-form-group>
              </span>
            </span>
          </b-form>
        </b-card>
      </b-card-group>
      <b-card-group :class="{ 'd-none': mode == 'runMode' && runMode == 'getResultDirectly'}">
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
  </b-container>
</template>

<script lang="ts">
  import { Component, Vue, Watch } from 'vue-property-decorator'
  import { } from 'bootstrap-vue/lib/components'
  import $ from 'jquery'
  import { API } from '../service'

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
    directRunScale: number = 6
    runSpeed: number = 1
    stepRunState: string = ''

    mounted () {
      this.chessBoard = this.creatBoard(this.boardScale)
    }

    @Watch('boardScale')
    boardScaleChange (val) {
      this.chessBoard = this.creatBoard(parseInt(val))
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
      console.log(this.chessBoard)
    }

    changeStepRunState (state: string) {
      this.stepRunState = state
    }

    @Watch('stepRunState')
    stepRunStateChange (val: string) {
    }

    check () {
      let payload = {
        'chessBoard': JSON.stringify(this.chessBoard)
      }
      $.post(API.check, payload, function (data, status) {
        console.log('data: ' + data + 'state: ' + status)
      })
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
