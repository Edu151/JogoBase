<template>
  <div class="game">
    <h1>Campo Minado</h1>
    <div class="board">
      <div
        v-for="(row, rowIndex) in board"
        :key="rowIndex"
        class="row"
      >
        <div
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          class="cell"
          :class="{ revealed: cell.revealed, mine: cell.mine }"
          @click="revealCell(rowIndex, colIndex)"
          @contextmenu.prevent="markCell(rowIndex, colIndex)"
        >
          {{ cell.revealed ? (cell.mine ? 'X' : cell.adjacentMines) : '' }}
        </div>
      </div>
    </div>
    <button @click="restart">Reiniciar</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      board: [],
      rows: 10,
      cols: 10,
      mines: 15,
      gameOver: false,
    };
  },
  mounted() {
    this.generateBoard();
  },
  methods: {
    generateBoard() {
      // Inicializar o tabuleiro
      this.board = Array(this.rows)
        .fill(null)
        .map(() =>
          Array(this.cols).fill({
            revealed: false,
            mine: false,
            adjacentMines: 0,
            marked: false,
          })
        );

      // Colocar as minas aleatoriamente
      let minesPlaced = 0;
      while (minesPlaced < this.mines) {
        const row = Math.floor(Math.random() * this.rows);
        const col = Math.floor(Math.random() * this.cols);

        if (!this.board[row][col].mine) {
          this.board[row][col].mine = true;
          minesPlaced++;
        }
      }

      // Calcular o número de minas adjacentes para cada célula
      for (let row = 0; row < this.rows; row++) {
        for (let col = 0; col < this.cols; col++) {
          if (!this.board[row][col].mine) {
            let count = 0;
            for (let dx = -1; dx <= 1; dx++) {
              for (let dy = -1; dy <= 1; dy++) {
                const newRow = row + dx;
                const newCol = col + dy;
                if (
                  newRow >= 0 &&
                  newRow < this.rows &&
                  newCol >= 0 &&
                  newCol < this.cols &&
                  this.board[newRow][newCol].mine
                ) {
                  count++;
                }
              }
            }
            this.board[row][col].adjacentMines = count;
          }
        }
      }
    },
    revealCell(row, col) {
      if (this.gameOver) return;

      const cell = this.board[row][col];
      if (cell.revealed || cell.marked) return;

      cell.revealed = true;

      if (cell.mine) {
        this.gameOver = true;
        alert('Você perdeu! Reinicie o jogo.');
        return;
      }

      if (cell.adjacentMines === 0) {
        this.revealEmptyCells(row, col);
      }
    },
    revealEmptyCells(row, col) {
      const queue = [{ row, col }];

      while (queue.length) {
        const { row, col } = queue.shift();

        for (let dx = -1; dx <= 1; dx++) {
          for (let dy = -1; dy <= 1; dy++) {
            const newRow = row + dx;
            const newCol = col + dy;
            if (
              newRow >= 0 &&
              newRow < this.rows &&
              newCol >= 0 &&
              newCol < this.cols &&
              !this.board[newRow][newCol].revealed &&
              !this.board[newRow][newCol].marked
            ) {
              this.board[newRow][newCol].revealed = true;
              if (this.board[newRow][newCol].adjacentMines === 0) {
                queue.push({ row: newRow, col: newCol });
              }
            }
          }
        }
      }
    },
    markCell(row, col) {
      if (this.gameOver) return;

      const cell = this.board[row][col];
      if (cell.revealed) return;

      cell.marked = !cell.marked;
    },
    restart() {
      this.gameOver = false;
      this.generateBoard();
    },
  },
};
</script>

<style scoped>
.game {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 40px;
}

.board {
  display: grid;
  grid-template-columns: repeat(10, 1fr);
}

.row {
  display: flex;
}

.cell {
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid #ccc;
  width: 30px;
  height: 30px;
  font-size: 20px;
  cursor: pointer;
  user-select: none;
}

.cell.revealed {
  background-color: #eee;
}

.cell.mine {
  background-color: red;
  color: white;
}

button {
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 18px;
}
</style>
