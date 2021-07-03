<template>
  <div style="text-align: center">
    <h2>
      <div>Цель набрать {{ needDial }} очков</div>
      <div>Ваши очки: {{ scores }}</div>
      <div>Ходов осталось: {{ movesLeft }}</div>
      <div>Перемешивания: {{ refresh }}</div>
    </h2>

    <div class="field" :style="fieldStyle">
      <span
        v-for="(column, colIndex) in blastArray"
        :key="colIndex + 'column'"
        class="column"
      >
        <span
          v-for="(line, lineIndex) in column"
          :key="lineIndex + 'line'"
          class="cell"
          :style="'background:' + getCellColor(colIndex, lineIndex)"
          @click="chose(colIndex, lineIndex)"
        />
      </span>
    </div>

    <button v-if="refresh > 0" @click="refreshField">
      Перемешать поле
    </button>
  </div>
</template>

<script>
export default {
  created() {
    const style = document.createElement("style");
    style.setAttribute("type", "text/css");
    document.head.appendChild(style);
    style.sheet.insertRule(`
      @keyframes showing {
        from {
          transform: translate(0, ${this.lineCount * 100}%);
        }
        to {
          transform: translate(0, 0);
        }
      }
    `);

    for (let i = 0; i < this.countColor; i++) {
      this.colors.push(this.createColor());
    }

    this.createField();
  },
  data() {
    return {
      countColor: 5,
      columnCount: 10,
      lineCount: 10,

      blastArray: [],
      colors: [],

      minInGroup: 2,
      movesLeft: 32,
      scores: 0,
      needDial: 200,
      refresh: 2,
    };
  },
  methods: {
    chose(column, line) {
      if (this.movesLeft > 0) {
        let group = this.getGroup(column, line);

        if (group.length > 1) {
          this.scores += group.length;
          this.movesLeft -= 1;

          group.forEach((item) => {
            this.blastArray[item.column][item.line] = "";
          });

          setTimeout(() => {
            this.blastArray.forEach((col, colIndex, blast) => {
              blast[colIndex] = blast[colIndex].filter((cell) => cell);
            });
          }, 200);

          setTimeout(() => {
            this.blastArray.forEach((col, colIndex, blast) => {
              const emptyLineCount = this.lineCount - blast[colIndex].length;

              for (let k = 0; k < emptyLineCount; k++) {
                blast[colIndex].push(this.getAvailableColor());
              }
            });
          }, 400);
        }
      }
    },
    getCellColor(column, line) {
      return this.blastArray[column][line];
    },
    getAvailableColor() {
      let color = this.colors[Math.floor(Math.random() * this.countColor)];

      return color;
    },
    createColor() {
      let randomColor = "#";
      let char = "0123456789abcdefghijklmnopqrstuvwxyz";

      for (let i = 0; i < 6; i++) {
        randomColor = randomColor + char[Math.floor(Math.random() * 16)];
      }

      if (this.colors.some((color) => color === randomColor)) {
        return this.createColor();
      } else return randomColor;
    },
    getGroup(column, line, arrGroup = [{ column, line }]) {
      const color = this.blastArray[column][line];

      if (
        line < this.lineCount - 1 &&
        color === this.blastArray[column][line + 1] &&
        !arrGroup.some(
          (item) => item.column === column && item.line === line + 1
        )
      ) {
        arrGroup.push({ column, line: line + 1 });
        arrGroup = this.getGroup(column, line + 1, arrGroup);
      }

      if (
        line > 0 &&
        color === this.blastArray[column][line - 1] &&
        !arrGroup.some(
          (item) => item.column === column && item.line === line - 1
        )
      ) {
        arrGroup.push({ column, line: line - 1 });
        arrGroup = this.getGroup(column, line - 1, arrGroup);
      }

      if (
        column < this.columnCount - 1 &&
        color === this.blastArray[column + 1][line] &&
        !arrGroup.some(
          (item) => item.column === column + 1 && item.line === line
        )
      ) {
        arrGroup.push({ column: column + 1, line });
        arrGroup = this.getGroup(column + 1, line, arrGroup);
      }

      if (
        column > 0 &&
        color === this.blastArray[column - 1][line] &&
        !arrGroup.some(
          (item) => item.column === column - 1 && item.line === line
        )
      ) {
        arrGroup.push({ column: column - 1, line });
        arrGroup = this.getGroup(column - 1, line, arrGroup);
      }

      return arrGroup;
    },
    createField() {
      this.blastArray = [];

      for (let column = 0; column < this.columnCount; column++) {
        let lineArray = [];

        for (let line = 0; line < this.lineCount; line++) {
          const color = this.getAvailableColor();
          lineArray.push(color);
        }

        this.blastArray.push(lineArray);
      }

      for (let group = 2; group <= this.minInGroup; group++) {
        for (let column = 0; column < this.columnCount; column++) {
          for (let line = 0; line < this.lineCount; line++) {
            while (this.getGroup(column, line).length < group) {
              this.blastArray[column][line] = this.getAvailableColor();
            }
          }
        }
      }
    },
    refreshField() {
      if (this.refresh > 0) {
        this.refresh -= 1;
        this.createField();
      }
    },
  },
  computed: {
    fieldStyle() {
      return {
        minHeight: 30 * this.lineCount + "px",
        minWidth: 30 * this.columnCount + "px",
        "grid-template-columns": `repeat(${this.columnCount}, 1fr)`,
      };
    },
  },
  watch: {
    movesLeft(val) {
      if (val === 0) {
        setTimeout(() => {
          this.scores >= this.needDial
            ? alert(`You win! \nВаши очки: ${this.scores}`)
            : alert(`Game Over! \nВаши очки: ${this.scores}`);
          location.reload();
        }, 1000);
      }
    },
  },
};
</script>

<style>
body {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: #373b44;
  background: -webkit-linear-gradient(to right, #373b44, #4286f4);
  background: linear-gradient(to right, #373b44, #4286f4);
  margin: 30px 0;
}

* {
  box-sizing: border-box;
}

.field {
  display: grid;

  /* display: flex;
  justify-content: center; */

  background: rgba(000, 000, 000, 0.2);
  border-radius: 5px;
  overflow: hidden;
  border: 2px solid black;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
  transform: rotate(180deg);
}

.column {
  display: flex;
  flex-direction: column;
}

.cell {
  display: block;
  width: 30px;
  height: 30px;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.2s ease-out;
  animation: showing 0.7s linear;
}

/* @keyframes showing {
  from {
    transform: translate(0, 1000%);
  }

  to {
    transform: translate(0, 0);
  }
} */
</style>
