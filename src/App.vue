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
      width: 20,
      height: 20,
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
      this.getRandomEmpty().color = this.colors.O;
    },

    drawInline() {
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

    nbLine(color) {
      return color === this.colors.X
        ? this.lineX.length + this.rowX.length
        : this.lineO.length + this.rowO.length;
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
