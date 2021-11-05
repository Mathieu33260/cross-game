<template>
  <div id="app">
    <input type="button" style="background: none" value="Reset" @click="reset">
    <input type="button" style="background: none" value="Zoom+" @click="increaseZoom">
    <input type="button" style="background: none" value="Zoom-" @click="decreaseZoom">
    <label for="height">Lines</label>
    <input type="number" id="height" style="background: none" :value="height" @click="changeHeight">
    <label for="width">Rows</label>
    <input type="number" id="width" style="background: none" :value="width" @click="changeWidth">

    <div style="display: flex">
      <div style="margin-right: 20px">
        Turn : {{ turn }}
      </div>
      <div style="margin-right: 20px">
        X : {{ nbLine(colors.X) }}
      </div>
      <div style="margin-right: 20px">
        O : {{ nbLine(colors.O) }}
      </div>
    </div>

    <div v-if="grid.length" style="display: flex; margin-top: 30px" id="grid">
      <RecycleScroller
          style="display: flex; align-self: center; width: 100%"
          :items="grid"
          :item-size="30"
          key-field="id"
          v-slot="{ item }"
      >
        <div style="display: flex; margin-top: -30px">
          <div
              v-for="cell in item.line"
              :style="'display: flex; justify-content: center; align-items: center; font-size: ' + size + 'px; cursor: pointer; width: ' + size + 'px; height: ' + size + 'px; border: 1px solid black;'"
              @click="selectCell(cell)"
              :class="cell.color === colors.X ? 'blue-font' : 'red-font'"
          >
            {{ cell.color !== colors.EMPTY ? cell.color : '' }}
            <div
                v-if="cell.right === true"
                :style="'height: 0px; border-bottom: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; border-top: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; position: absolute; left: 50%; width: 50%'"
            ></div>
            <div
                v-if="cell.left === true"
                :style="'height: 0px; border-bottom: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; border-top: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; position: absolute; right: 50%; width: 50%'"
            ></div>
            <div
                v-if="cell.bottom === true"
                :style="'width: 0px; border-left: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; border-right: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; position: absolute; top: 50%; height: 50%'"
            ></div>
            <div
                v-if="cell.top === true"
                :style="'width: 0px; border-left: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; border-right: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; position: absolute; bottom: 50%; height: 50%'"
            ></div>
            <div
                v-if="cell.topRight === true"
                :style="'width: 0px; border-left: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; border-right: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; position: absolute; bottom: 45%; left: 70%; height: 65%; transform: rotate(45deg);'"
            ></div>
            <div
                v-if="cell.topLeft === true"
                :style="'width: 0px; border-left: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; border-right: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; position: absolute; bottom: 45%; right: 70%; height: 65%; transform: rotate(-45deg);'"
            ></div>
            <div
                v-if="cell.bottomLeft === true"
                :style="'width: 0px; border-left: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; border-right: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; position: absolute; top: 45%; right: 70%; height: 65%; transform: rotate(-135deg);'"
            ></div>
            <div
                v-if="cell.bottomRight === true"
                :style="'width: 0px; border-left: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; border-right: 1px solid '
                + (cell.color === colors.X ? 'blue' : 'red') + '; position: absolute; top: 45%; left: 70%; height: 65%; transform: rotate(135deg);'"
            ></div>
          </div>
        </div>
      </RecycleScroller>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {
  },

  data() {
    return {
      grid: [],
      width: 4,
      height: 4,
      displayed: null,
      interval: null,
      speed: 1000,
      running: false,
      turn: 0,
      size: 20,
      colors: {
        X: 'X',
        O: 'O',
        EMPTY: 'EMPTY',
      },
      lineX: [],
      lineO: [],
      rowX: [],
      rowO: [],
      diagRightX: [],
      diagRightO: [],
      diagLeftX: [],
      diagLeftO: [],
      max_depth: 3,
      nodes_map: new Map(),
    }
  },

  created() {
    this.reset();
    history.pushState(null, null, location.href);
    window.onpopstate = function () {
      history.go(1);
    };
  },

  methods: {

    selectCell(cell) {
      if (cell.color === this.colors.EMPTY) {
        cell.color = this.colors.X;
        this.computerPlay();
        this.drawInline();
        this.turn ++;
      }
    },

    computerPlay() {
      //this.getRandomEmpty().color = this.colors.O;
      const id = this.getBestMove(
          this.grid,
          this.lineX,
          this.lineO,
          this.rowX,
          this.rowO,
          this.diagRightX,
          this.diagRightO,
          this.diagLeftX,
          this.diagLeftO,
      );
      //console.log(this.getCellById(parseInt(id, 10)))//.color = this.colors.O;
      //console.log(typeof id)//.color = this.colors.O;
      this.getCellById(parseInt(id, 10)).color = this.colors.O;
    },

    drawInline(
        grid = null,
        lineX = null,
        lineO = null,
        rowX = null,
        rowO = null,
        diagRightX = null,
        diagRightO = null,
        diagLeftX = null,
        diagLeftO = null,
    ) {
      if (grid !== null) {
        grid.forEach((l, li) => {
          l.line.forEach((c, ci) => {
            //////////////
            if (ci + 3 < this.width) {
              if (grid[li].line[ci].color === this.colors.X
                  && grid[li].line[ci + 1].color === this.colors.X
                  && grid[li].line[ci + 2].color === this.colors.X
                  && grid[li].line[ci + 3].color === this.colors.X
                  && (
                      lineX.some(l => l.includes(grid[li].line[ci + 1].id) === true)
                      || lineX.some(l => l.includes(grid[li].line[ci + 2].id) === true)
                  ) === false
              ) {

                lineX.push([
                  c.id,
                  grid[li].line[ci + 1].id,
                  grid[li].line[ci + 2].id,
                  grid[li].line[ci + 3].id,
                ]);
                c.right = true;
                grid[li].line[ci + 1].left = true;
                grid[li].line[ci + 1].right = true;
                grid[li].line[ci + 2].left = true;
                grid[li].line[ci + 2].right = true;
                grid[li].line[ci + 3].left = true;

              } else if (
                  grid[li].line[ci].color === this.colors.O
                  && grid[li].line[ci + 1].color === this.colors.O
                  && grid[li].line[ci + 2].color === this.colors.O
                  && grid[li].line[ci + 3].color === this.colors.O
                  && (
                      lineO.some(l => l.includes(grid[li].line[ci + 1].id) === true)
                      || lineO.some(l => l.includes(grid[li].line[ci + 2].id) === true)
                  ) === false
              ) {

                lineO.push([
                  c.id,
                  grid[li].line[ci + 1].id,
                  grid[li].line[ci + 2].id,
                  grid[li].line[ci + 3].id,
                ]);
                c.right = true;
                grid[li].line[ci + 1].left = true;
                grid[li].line[ci + 1].right = true;
                grid[li].line[ci + 2].left = true;
                grid[li].line[ci + 2].right = true;
                grid[li].line[ci + 3].left = true;
              }
            }
            //////////////
            if (li + 3 < this.height) {
              if (grid[li].line[ci].color === this.colors.X
                  && grid[li + 1].line[ci].color === this.colors.X
                  && grid[li + 2].line[ci].color === this.colors.X
                  && grid[li + 3].line[ci].color === this.colors.X
                  && (
                      rowX.some(l => l.includes(grid[li + 1].line[ci].id) === true)
                      || rowX.some(l => l.includes(grid[li + 2].line[ci].id) === true)
                  ) === false
              ) {

                rowX.push([
                  c.id,
                  grid[li + 1].line[ci].id,
                  grid[li + 2].line[ci].id,
                  grid[li + 3].line[ci].id,
                ]);
                c.bottom = true;
                grid[li + 1].line[ci].top = true;
                grid[li + 1].line[ci].bottom = true;
                grid[li + 2].line[ci].top = true;
                grid[li + 2].line[ci].bottom = true;
                grid[li + 3].line[ci].top = true;

              } else if (grid[li].line[ci].color === this.colors.O
                  && grid[li + 1].line[ci].color === this.colors.O
                  && grid[li + 2].line[ci].color === this.colors.O
                  && grid[li + 3].line[ci].color === this.colors.O
                  && (
                      rowO.some(l => l.includes(grid[li + 1].line[ci].id) === true)
                      || rowO.some(l => l.includes(grid[li + 2].line[ci].id) === true)
                  ) === false
              ) {

                rowO.push([
                  c.id,
                  grid[li + 1].line[ci].id,
                  grid[li + 2].line[ci].id,
                  grid[li + 3].line[ci].id,
                ]);
                c.bottom = true;
                grid[li + 1].line[ci].top = true;
                grid[li + 1].line[ci].bottom = true;
                grid[li + 2].line[ci].top = true;
                grid[li + 2].line[ci].bottom = true;
                grid[li + 3].line[ci].top = true;
              }
            }
            //////////////
            if (li + 3 < this.height && ci + 3 < this.width) {
              if (grid[li].line[ci].color === this.colors.X
                  && grid[li + 1].line[ci + 1].color === this.colors.X
                  && grid[li + 2].line[ci + 2].color === this.colors.X
                  && grid[li + 3].line[ci + 3].color === this.colors.X
                  && (
                      diagRightX.some(l => l.includes(grid[li + 1].line[ci + 1].id) === true)
                      || diagRightX.some(l => l.includes(grid[li + 2].line[ci + 2].id) === true)
                  ) === false
              ) {

                diagRightX.push([
                  c.id,
                  grid[li + 1].line[ci + 1].id,
                  grid[li + 2].line[ci + 2].id,
                  grid[li + 3].line[ci + 3].id,
                ]);
                c.bottomRight = true;
                grid[li + 1].line[ci + 1].topLeft = true;
                grid[li + 1].line[ci + 1].bottomRight = true;
                grid[li + 2].line[ci + 2].topLeft = true;
                grid[li + 2].line[ci + 2].bottomRight = true;
                grid[li + 3].line[ci + 3].topLeft = true;

              } else if (grid[li].line[ci].color === this.colors.O
                  && grid[li + 1].line[ci + 1].color === this.colors.O
                  && grid[li + 2].line[ci + 2].color === this.colors.O
                  && grid[li + 3].line[ci + 3].color === this.colors.O
                  && (
                      diagRightO.some(l => l.includes(grid[li + 1].line[ci + 1].id) === true)
                      || diagRightO.some(l => l.includes(grid[li + 2].line[ci + 2].id) === true)
                  ) === false
              ) {

                diagRightO.push([
                  c.id,
                  grid[li + 1].line[ci + 1].id,
                  grid[li + 2].line[ci + 2].id,
                  grid[li + 3].line[ci + 3].id,
                ]);
                c.bottomRight = true;
                grid[li + 1].line[ci + 1].topLeft = true;
                grid[li + 1].line[ci + 1].bottomRight = true;
                grid[li + 2].line[ci + 2].topLeft = true;
                grid[li + 2].line[ci + 2].bottomRight = true;
                grid[li + 3].line[ci + 3].topLeft = true;
              }
            }
            //////////
            if (li + 3 < this.height && ci - 3 >= 0) {
              if (grid[li].line[ci].color === this.colors.X
                  && grid[li + 1].line[ci - 1].color === this.colors.X
                  && grid[li + 2].line[ci - 2].color === this.colors.X
                  && grid[li + 3].line[ci - 3].color === this.colors.X
                  && (
                      diagLeftX.some(l => l.includes(grid[li + 1].line[ci - 1].id) === true)
                      || diagLeftX.some(l => l.includes(grid[li + 2].line[ci - 2].id) === true)
                  ) === false
              ) {

                diagLeftX.push([
                  c.id,
                  grid[li + 1].line[ci - 1].id,
                  grid[li + 2].line[ci - 2].id,
                  grid[li + 3].line[ci - 3].id,
                ]);
                c.bottomLeft = true;
                grid[li + 1].line[ci - 1].topRight = true;
                grid[li + 1].line[ci - 1].bottomLeft = true;
                grid[li + 2].line[ci - 2].topRight = true;
                grid[li + 2].line[ci - 2].bottomLeft = true;
                grid[li + 3].line[ci - 3].topRight = true;

              } else if (grid[li].line[ci].color === this.colors.O
                  && grid[li + 1].line[ci - 1].color === this.colors.O
                  && grid[li + 2].line[ci - 2].color === this.colors.O
                  && grid[li + 3].line[ci - 3].color === this.colors.O
                  && (
                      diagLeftO.some(l => l.includes(grid[li + 1].line[ci - 1].id) === true)
                      || diagLeftO.some(l => l.includes(grid[li + 2].line[ci - 2].id) === true)
                  ) === false
              ) {

                diagLeftO.push([
                  c.id,
                  grid[li + 1].line[ci - 1].id,
                  grid[li + 2].line[ci - 2].id,
                  grid[li + 3].line[ci - 3].id,
                ]);
                c.bottomLeft = true;
                grid[li + 1].line[ci - 1].topRight = true;
                grid[li + 1].line[ci - 1].bottomLeft = true;
                grid[li + 2].line[ci - 2].topRight = true;
                grid[li + 2].line[ci - 2].bottomLeft = true;
                grid[li + 3].line[ci - 3].topRight = true;
              }
            }
          })
        })
        return {
          grid: grid,
          lineX: lineX,
          lineO: lineO,
          rowX: rowX,
          rowO: rowO,
          diagRightX: diagRightX,
          diagRightO: diagRightO,
          diagLeftX: diagLeftX,
          diagLeftO: diagLeftO,
        }
      } else {
        this.grid.forEach((l, li) => {
          l.line.forEach((c, ci) => {
            //////////////
            if (ci + 3 < this.width) {
              if (this.grid[li].line[ci].color === this.colors.X
                  && this.grid[li].line[ci + 1].color === this.colors.X
                  && this.grid[li].line[ci + 2].color === this.colors.X
                  && this.grid[li].line[ci + 3].color === this.colors.X
                  && (
                      this.lineX.some(l => l.includes(this.grid[li].line[ci + 1].id) === true)
                      || this.lineX.some(l => l.includes(this.grid[li].line[ci + 2].id) === true)
                  ) === false
              ) {

                this.lineX.push([
                  c.id,
                  this.grid[li].line[ci + 1].id,
                  this.grid[li].line[ci + 2].id,
                  this.grid[li].line[ci + 3].id,
                ]);
                c.right = true;
                this.grid[li].line[ci + 1].left = true;
                this.grid[li].line[ci + 1].right = true;
                this.grid[li].line[ci + 2].left = true;
                this.grid[li].line[ci + 2].right = true;
                this.grid[li].line[ci + 3].left = true;

              } else if (
                  this.grid[li].line[ci].color === this.colors.O
                  && this.grid[li].line[ci + 1].color === this.colors.O
                  && this.grid[li].line[ci + 2].color === this.colors.O
                  && this.grid[li].line[ci + 3].color === this.colors.O
                  && (
                      this.lineO.some(l => l.includes(this.grid[li].line[ci + 1].id) === true)
                      || this.lineO.some(l => l.includes(this.grid[li].line[ci + 2].id) === true)
                  ) === false
              ) {

                this.lineO.push([
                  c.id,
                  this.grid[li].line[ci + 1].id,
                  this.grid[li].line[ci + 2].id,
                  this.grid[li].line[ci + 3].id,
                ]);
                c.right = true;
                this.grid[li].line[ci + 1].left = true;
                this.grid[li].line[ci + 1].right = true;
                this.grid[li].line[ci + 2].left = true;
                this.grid[li].line[ci + 2].right = true;
                this.grid[li].line[ci + 3].left = true;
              }
            }
            //////////////
            if (li + 3 < this.height) {
              if (this.grid[li].line[ci].color === this.colors.X
                  && this.grid[li + 1].line[ci].color === this.colors.X
                  && this.grid[li + 2].line[ci].color === this.colors.X
                  && this.grid[li + 3].line[ci].color === this.colors.X
                  && (
                      this.rowX.some(l => l.includes(this.grid[li + 1].line[ci].id) === true)
                      || this.rowX.some(l => l.includes(this.grid[li + 2].line[ci].id) === true)
                  ) === false
              ) {

                this.rowX.push([
                  c.id,
                  this.grid[li + 1].line[ci].id,
                  this.grid[li + 2].line[ci].id,
                  this.grid[li + 3].line[ci].id,
                ]);
                c.bottom = true;
                this.grid[li + 1].line[ci].top = true;
                this.grid[li + 1].line[ci].bottom = true;
                this.grid[li + 2].line[ci].top = true;
                this.grid[li + 2].line[ci].bottom = true;
                this.grid[li + 3].line[ci].top = true;

              } else if (this.grid[li].line[ci].color === this.colors.O
                  && this.grid[li + 1].line[ci].color === this.colors.O
                  && this.grid[li + 2].line[ci].color === this.colors.O
                  && this.grid[li + 3].line[ci].color === this.colors.O
                  && (
                      this.rowO.some(l => l.includes(this.grid[li + 1].line[ci].id) === true)
                      || this.rowO.some(l => l.includes(this.grid[li + 2].line[ci].id) === true)
                  ) === false
              ) {

                this.rowO.push([
                  c.id,
                  this.grid[li + 1].line[ci].id,
                  this.grid[li + 2].line[ci].id,
                  this.grid[li + 3].line[ci].id,
                ]);
                c.bottom = true;
                this.grid[li + 1].line[ci].top = true;
                this.grid[li + 1].line[ci].bottom = true;
                this.grid[li + 2].line[ci].top = true;
                this.grid[li + 2].line[ci].bottom = true;
                this.grid[li + 3].line[ci].top = true;
              }
            }
            //////////////
            if (li + 3 < this.height && ci + 3 < this.width) {
              if (this.grid[li].line[ci].color === this.colors.X
                  && this.grid[li + 1].line[ci + 1].color === this.colors.X
                  && this.grid[li + 2].line[ci + 2].color === this.colors.X
                  && this.grid[li + 3].line[ci + 3].color === this.colors.X
                  && (
                      this.diagRightX.some(l => l.includes(this.grid[li + 1].line[ci + 1].id) === true)
                      || this.diagRightX.some(l => l.includes(this.grid[li + 2].line[ci + 2].id) === true)
                  ) === false
              ) {

                this.diagRightX.push([
                  c.id,
                  this.grid[li + 1].line[ci + 1].id,
                  this.grid[li + 2].line[ci + 2].id,
                  this.grid[li + 3].line[ci + 3].id,
                ]);
                c.bottomRight = true;
                this.grid[li + 1].line[ci + 1].topLeft = true;
                this.grid[li + 1].line[ci + 1].bottomRight = true;
                this.grid[li + 2].line[ci + 2].topLeft = true;
                this.grid[li + 2].line[ci + 2].bottomRight = true;
                this.grid[li + 3].line[ci + 3].topLeft = true;

              } else if (this.grid[li].line[ci].color === this.colors.O
                  && this.grid[li + 1].line[ci + 1].color === this.colors.O
                  && this.grid[li + 2].line[ci + 2].color === this.colors.O
                  && this.grid[li + 3].line[ci + 3].color === this.colors.O
                  && (
                      this.diagRightO.some(l => l.includes(this.grid[li + 1].line[ci + 1].id) === true)
                      || this.diagRightO.some(l => l.includes(this.grid[li + 2].line[ci + 2].id) === true)
                  ) === false
              ) {

                this.diagRightO.push([
                  c.id,
                  this.grid[li + 1].line[ci + 1].id,
                  this.grid[li + 2].line[ci + 2].id,
                  this.grid[li + 3].line[ci + 3].id,
                ]);
                c.bottomRight = true;
                this.grid[li + 1].line[ci + 1].topLeft = true;
                this.grid[li + 1].line[ci + 1].bottomRight = true;
                this.grid[li + 2].line[ci + 2].topLeft = true;
                this.grid[li + 2].line[ci + 2].bottomRight = true;
                this.grid[li + 3].line[ci + 3].topLeft = true;
              }
            }
            //////////
            if (li + 3 < this.height && ci - 3 >= 0) {
              if (this.grid[li].line[ci].color === this.colors.X
                  && this.grid[li + 1].line[ci - 1].color === this.colors.X
                  && this.grid[li + 2].line[ci - 2].color === this.colors.X
                  && this.grid[li + 3].line[ci - 3].color === this.colors.X
                  && (
                      this.diagLeftX.some(l => l.includes(this.grid[li + 1].line[ci - 1].id) === true)
                      || this.diagLeftX.some(l => l.includes(this.grid[li + 2].line[ci - 2].id) === true)
                  ) === false
              ) {

                this.diagLeftX.push([
                  c.id,
                  this.grid[li + 1].line[ci - 1].id,
                  this.grid[li + 2].line[ci - 2].id,
                  this.grid[li + 3].line[ci - 3].id,
                ]);
                c.bottomLeft = true;
                this.grid[li + 1].line[ci - 1].topRight = true;
                this.grid[li + 1].line[ci - 1].bottomLeft = true;
                this.grid[li + 2].line[ci - 2].topRight = true;
                this.grid[li + 2].line[ci - 2].bottomLeft = true;
                this.grid[li + 3].line[ci - 3].topRight = true;

              } else if (this.grid[li].line[ci].color === this.colors.O
                  && this.grid[li + 1].line[ci - 1].color === this.colors.O
                  && this.grid[li + 2].line[ci - 2].color === this.colors.O
                  && this.grid[li + 3].line[ci - 3].color === this.colors.O
                  && (
                      this.diagLeftO.some(l => l.includes(this.grid[li + 1].line[ci - 1].id) === true)
                      || this.diagLeftO.some(l => l.includes(this.grid[li + 2].line[ci - 2].id) === true)
                  ) === false
              ) {

                this.diagLeftO.push([
                  c.id,
                  this.grid[li + 1].line[ci - 1].id,
                  this.grid[li + 2].line[ci - 2].id,
                  this.grid[li + 3].line[ci - 3].id,
                ]);
                c.bottomLeft = true;
                this.grid[li + 1].line[ci - 1].topRight = true;
                this.grid[li + 1].line[ci - 1].bottomLeft = true;
                this.grid[li + 2].line[ci - 2].topRight = true;
                this.grid[li + 2].line[ci - 2].bottomLeft = true;
                this.grid[li + 3].line[ci - 3].topRight = true;
              }
            }
          })
        })
      }
      return null;
    },

    isTerminal(grid) {
      return grid.reduce((acc, current) => acc + (current.line.reduce((acc2, current2) => acc2 + (current2.color === this.colors.EMPTY ? 1 : 0), 0)), 0) === 0;
    },

    getAvailableMoves(grid) {
      const moves = [];
      grid.forEach((line) => {
        line.line.forEach((cell) => {
          if (cell.color === this.colors.EMPTY) {
            moves.push(cell);
          }
        })
      });
      return moves;
    },

    getBestMove(
        grid = null,
        lineX = null,
        lineO = null,
        rowX = null,
        rowO = null,
        diagRightX = null,
        diagRightO = null,
        diagLeftX = null,
        diagLeftO = null,
        maximizing = true,
        callback = () => {},
        depth = 0
    ) {

      //clear nodes_map if the function is called for a new move
      if(depth === 0) {
        this.nodes_map.clear();
      }

      //If the board state is a terminal one, return the heuristic value
      if(this.isTerminal(grid) || depth === this.max_depth ) {
        const scoreO = this.nbLine(
            this.colors.O,
            grid,
            lineX,
            lineO,
            rowX,
            rowO,
            diagRightX,
            diagRightO,
            diagLeftX,
            diagLeftO,
        );
        const scoreX = this.nbLine(
            this.colors.X,
            grid,
            lineX,
            lineO,
            rowX,
            rowO,
            diagRightX,
            diagRightO,
            diagLeftX,
            diagLeftO,
        );
        console.log(scoreO - scoreX + ' depth: ' + depth)
        let score = scoreO - scoreX;
        return score > 0 ? score - depth : score + depth;
      }

      if(maximizing) {
        const scoreO = this.nbLine(
            this.colors.O,
            grid,
            lineX,
            lineO,
            rowX,
            rowO,
            diagRightX,
            diagRightO,
            diagLeftX,
            diagLeftO,
        );
        const scoreX = this.nbLine(
            this.colors.X,
            grid,
            lineX,
            lineO,
            rowX,
            rowO,
            diagRightX,
            diagRightO,
            diagLeftX,
            diagLeftO,
        );
        let best = scoreO - scoreX;

        //Loop through all empty cells
        this.getAvailableMoves(grid).forEach(cell => {
          //Initialize a new board with the current state (slice() is used to create a new array and not modify the original)
          let child = JSON.parse(JSON.stringify(grid));
          lineX = JSON.parse(JSON.stringify(lineX));
          lineO = JSON.parse(JSON.stringify(lineO));
          rowX = JSON.parse(JSON.stringify(rowX));
          rowO = JSON.parse(JSON.stringify(rowO));
          diagRightX = JSON.parse(JSON.stringify(diagRightX));
          diagRightO = JSON.parse(JSON.stringify(diagRightO));
          diagLeftX = JSON.parse(JSON.stringify(diagLeftX));
          diagLeftO = JSON.parse(JSON.stringify(diagLeftO));

          //Create a child node by inserting the maximizing symbol x into the current emoty cell
          child[cell.y].line[cell.x].color = this.colors.X;

          const newLines = this.drawInline(
              child,
              lineX,
              lineO,
              rowX,
              rowO,
              diagRightX,
              diagRightO,
              diagLeftX,
              diagLeftO
          );

          //Recursively calling getBestMove this time with the new board and minimizing turn and incrementing the depth
          let node_value = this.getBestMove(
              newLines.grid,
              newLines.lineX,
              newLines.lineO,
              newLines.rowX,
              newLines.rowO,
              newLines.diagRightX,
              newLines.diagRightO,
              newLines.diagLeftX,
              newLines.diagLeftO,
              false,
              callback,
              depth + 1
          );

          //Updating best value
          best = Math.max(best, node_value);

          //If it's the main function call, not a recursive one, map each heuristic value with it's moves indicies
          if(depth === 0) {
            //Comma seperated indicies if multiple moves have the same heuristic value
            let moves = this.nodes_map.has(node_value) ? `${this.nodes_map.get(node_value)},${cell.id}` : cell.id;
            this.nodes_map.set(node_value, moves);
          }
        });
        //If it's the main call, return the index of the best move or a random index if multiple indicies have the same value
        if(depth === 0) {
          let ret;
          if(typeof this.nodes_map.get(best) === 'string') {
            let arr = this.nodes_map.get(best).split(',');
            let rand = Math.floor(Math.random() * arr.length);
            ret = arr[rand];
          } else {
            ret = this.nodes_map.get(best);
          }
          //run a callback after calculation and return the index
          callback(ret);
          return ret;
        }
        //If not main call (recursive) return the heuristic value for next calculation
        return best;
      }

      if(!maximizing) {
        const scoreO = this.nbLine(
            this.colors.O,
            grid,
            lineX,
            lineO,
            rowX,
            rowO,
            diagRightX,
            diagRightO,
            diagLeftX,
            diagLeftO,
        );
        const scoreX = this.nbLine(
            this.colors.X,
            grid,
            lineX,
            lineO,
            rowX,
            rowO,
            diagRightX,
            diagRightO,
            diagLeftX,
            diagLeftO,
        );
        let best = scoreO - scoreX;
        //Loop through all empty cells
        this.getAvailableMoves(grid).forEach(cell => {
          //Initialize a new board with the current state (slice() is used to create a new array and not modify the original)
          let child = JSON.parse(JSON.stringify(grid));
          lineX = JSON.parse(JSON.stringify(lineX));
          lineO = JSON.parse(JSON.stringify(lineO));
          rowX = JSON.parse(JSON.stringify(rowX));
          rowO = JSON.parse(JSON.stringify(rowO));
          diagRightX = JSON.parse(JSON.stringify(diagRightX));
          diagRightO = JSON.parse(JSON.stringify(diagRightO));
          diagLeftX = JSON.parse(JSON.stringify(diagLeftX));
          diagLeftO = JSON.parse(JSON.stringify(diagLeftO));

          //Create a child node by inserting the maximizing symbol x into the current emoty cell
          child[cell.y].line[cell.x].color = this.colors.O;

          const newLines = this.drawInline(
              child,
              lineX,
              lineO,
              rowX,
              rowO,
              diagRightX,
              diagRightO,
              diagLeftX,
              diagLeftO
          );

          //Recursively calling getBestMove this time with the new board and minimizing turn and incrementing the depth
          let node_value = this.getBestMove(
              newLines.grid,
              newLines.lineX,
              newLines.lineO,
              newLines.rowX,
              newLines.rowO,
              newLines.diagRightX,
              newLines.diagRightO,
              newLines.diagLeftX,
              newLines.diagLeftO,
              true,
              callback,
              depth + 1
          );
          //Updating best value
          best = Math.min(best, node_value);

          //If it's the main function call, not a recursive one, map each heuristic value with it's moves indicies
          if(depth === 0) {
            //Comma seperated indicies if multiple moves have the same heuristic value
            let moves = this.nodes_map.has(node_value) ? `${this.nodes_map.get(node_value)},${cell.id}` : cell.id;
            this.nodes_map.set(node_value, moves);
          }
        });
        //If it's the main call, return the index of the best move or a random index if multiple indicies have the same value
        if(depth === 0) {
          let ret;
          if(typeof this.nodes_map.get(best) === 'string') {
            let arr = this.nodes_map.get(best).split(',');
            let rand = Math.floor(Math.random() * arr.length);
            ret = arr[rand];
          } else {
            ret = this.nodes_map.get(best);
          }
          //run a callback after calculation and return the index
          callback(ret);
          return ret;
        }
        //If not main call (recursive) return the heuristic value for next calculation
        return best;
      }
    },

    getRandomEmpty() {
      let cell;
      this.grid.every(l => {
        return l.line.every(c => {
          if (c.color === this.colors.EMPTY) {
            cell = c;
            return false;
          }
          return true;
        })
      })
      return cell;
    },

    getCellById(id) {
      let cell;
      this.grid.every(l => {
        return l.line.every(c => {
          if (c.id === id) {
            cell = c;
            return false;
          }
          return true;
        })
      })
      return cell;
    },

    increaseZoom() {
      this.size++;
    },

    decreaseZoom() {
      this.size--;
    },

    changeHeight(event) {
      this.height = parseInt(event.target.value, 10);
      this.reset();
    },

    changeWidth(event) {
      this.width = parseInt(event.target.value, 10);
      this.reset();
    },

    increaseSpeed() {
      this.speed /= 2;
      this.clearAndRelaunch();
    },

    decreaseSpeed() {
      this.speed *= 2;
      this.clearAndRelaunch();
    },

    clearAndRelaunch() {
      if (this.running) {
        clearInterval(this.interval);
        this.$nextTick(() => {
          this.run();
        })
      }
    },

    stop() {
      this.running = false;
      clearInterval(this.interval);
    },

    run() {
      this.running = true;
      this.interval = setInterval(() => {
        this.updateGrid();
      }, this.speed);
    },

    getColor(y, x) {
      if (y === 0 && x === 0) {
        return this.colors.BLUE;
      }
      if (y === this.height - 1 && x === this.width - 1) {
        return this.colors.RED;
      }
      return this.getRandomInt(3) % 3 === 0 ? this.colors.BLACK : this.colors.WHITE
    },

    reset() {
      this.turn = 0;
      this.lineO = [];
      this.lineX = [];
      this.rowO = [];
      this.rowX = [];
      this.diagLeftO = [];
      this.diagLeftX = [];
      this.diagRightX = [];
      this.diagRightO = [];
      this.stop();
      this.grid = [];
      let id = 0;
      for (let i = 0; i < this.height; i++) {
        let line = [];
        for (let j = 0; j < this.width; j++) {
          line.push({
            id: id,
            x: j,
            y: i,
            color: this.colors.EMPTY,
          })
          id ++;
        }
        this.grid.push({
          id: i,
          line: line,
        });
      }
    },

    getRandomInt(max) {
      return Math.floor(Math.random() * Math.floor(max));
    },

    updateGrid() {
      this.turn ++;
      let tmpGrid = [];
      this.grid.forEach((line, indexLine) => {
        let tmpLine = [];
        line.line.forEach((cell, indexCell) => {
          let tmpCell = {};
          let nbAliveBlackAround = this.getNbAliveAround(indexLine, indexCell);

          tmpCell.color = cell.color;
          if (cell.color === this.colors.BLACK) {
            if (nbAliveBlackAround === 2 || nbAliveBlackAround === 3) {
              tmpCell.color = this.colors.BLACK;
            } else {
              tmpCell.color = this.colors.WHITE;
            }
          } /*else if (cell.color === this.colors.WHITE) {
            if (nbAliveAround === 3) {
              tmpCell.color = this.colors.BLACK;
            } else {
              tmpCell.color = this.colors.WHITE;
            }
          } else*/ if (cell.color === this.colors.WHITE) {
            let nbAliveBlueAround = this.getNbAliveAround(indexLine, indexCell, this.colors.BLUE);
            let nbAliveRedAround = this.getNbAliveAround(indexLine, indexCell, this.colors.RED);
            let probaBlue = nbAliveBlueAround * 0.1;
            let probaRed = nbAliveRedAround * 0.1;
            let rand = Math.round(Math.random() * 10) / 10;
            if (probaBlue > rand && probaBlue > probaRed) {
              tmpCell.color = this.colors.BLUE;
            } else if (probaRed > rand && probaBlue < probaRed) {
              tmpCell.color = this.colors.RED;
            } else {
              if (nbAliveBlackAround === 3) {
                tmpCell.color = this.colors.BLACK;
              } else {
                tmpCell.color = this.colors.WHITE;
              }
            }
          }

          tmpCell.x = cell.x;
          tmpCell.y = cell.y;
          tmpCell.id = cell.id;

          tmpLine.push(tmpCell);
        })
        tmpGrid.push({
          id: line.id,
          line: tmpLine,
        });
      })

      this.grid = tmpGrid;
    },

    getNbAliveAround(indexLine, indexCell, color = this.colors.BLACK) {
      let nbAliveAround = 0;

      if (this.grid[indexLine - 1]) {
        if (this.grid[indexLine - 1].line[indexCell - 1] && this.grid[indexLine - 1].line[indexCell - 1].color === color) {
          nbAliveAround++;
        }
        if (this.grid[indexLine - 1].line[indexCell] && this.grid[indexLine - 1].line[indexCell].color === color) {
          nbAliveAround++;
        }
        if (this.grid[indexLine - 1].line[indexCell + 1] && this.grid[indexLine - 1].line[indexCell + 1].color === color) {
          nbAliveAround++;
        }
      }

      if (this.grid[indexLine + 1]) {
        if (this.grid[indexLine + 1].line[indexCell - 1] && this.grid[indexLine + 1].line[indexCell - 1].color === color) {
          nbAliveAround++;
        }
        if (this.grid[indexLine + 1].line[indexCell] && this.grid[indexLine + 1].line[indexCell].color === color) {
          nbAliveAround++;
        }
        if (this.grid[indexLine + 1].line[indexCell + 1] && this.grid[indexLine + 1].line[indexCell + 1].color === color) {
          nbAliveAround++;
        }
      }

      if (this.grid[indexLine].line[indexCell - 1] && this.grid[indexLine].line[indexCell - 1].color === color) {
        nbAliveAround++;
      }
      if (this.grid[indexLine].line[indexCell + 1] && this.grid[indexLine].line[indexCell + 1].color === color) {
        nbAliveAround++;
      }

      return nbAliveAround;
    },

    nbColor(color) {
      return this.grid.reduce((acc, current) => acc + (current.line.reduce((acc2, current2) => acc2 + (current2.color === color ? 1 : 0), 0)), 0)
    },

    nbLine(
        color,
        grid = null,
        lineX = null,
        lineO = null,
        rowX = null,
        rowO = null,
        diagRightX = null,
        diagRightO = null,
        diagLeftX = null,
        diagLeftO = null,
    ) {
      if (grid !== null) {
        return color === this.colors.X
            ? lineX.length + rowX.length + diagRightX.length + diagLeftX.length
            : lineO.length + rowO.length + diagRightO.length + diagLeftO.length;
      }
      return color === this.colors.X
        ? this.lineX.length + this.rowX.length + this.diagRightX.length + this.diagLeftX.length
        : this.lineO.length + this.rowO.length + this.diagRightO.length + this.diagLeftO.length;
    },
  },

  computed: {



  },

  watch: {

  }
}
</script>

<style>
.blue-font {
  font-weight: 300;
  color: #6dcaef;
  position: relative;
}

.red-font {
  font-weight: 300;
  color: #ff7474;
  position: relative;
}
</style>
