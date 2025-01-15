<template id="BingoTem">
    <div class="bingo-container">
      <h3>Bingo Game</h3>
      <div class="bingo-board">
        <div
          v-for="(cell, index) in boardFlat"
          :key="index"
          :class="['bingo-cell', { selected: cell.selected, bingo: cell.isBingo }]"
          :style="{ backgroundColor: cell.isBingo ? 'rgba(124, 76, 220, 1)' : cell.selected ? 'rgba(109, 45, 8, 1)' : 'lightgray' }"
          @click="markCell(index)"
        >
          {{ cell.number }}
        </div>
        <!-- 绘制所有胜利的线 -->
        <div
          v-for="(style, index) in bingoLines"
          :key="index"
          class="bingo-line"
          :style="style"
        ></div>
      </div>
      <button @click="resetGame">Reset Game</button>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue';
  
  const size = 5; // 5x5 Bingo board
  const boardFlat = ref([]); // One-dimensional array to store board cells
  const bingo = ref(false); // Bingo flag
  const bingoLines = ref([]); // Array to store styles for each winning line
  
  // Initialize the Bingo board
  const initBoard = () => {
    let numbers = Array.from({ length: size * size }, (_, i) => i + 1);
    numbers = shuffleArray(numbers);
    boardFlat.value = numbers.map(number => ({ number, selected: false, isBingo: false }));
  };
  
  // Shuffle the array of numbers
  const shuffleArray = (array) => {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  };
  
  // Mark a cell as selected
  const markCell = (index) => {
    if (boardFlat.value[index].selected) return; // Skip if already selected
    boardFlat.value[index].selected = true;
    checkBingo();
  };
  
  // Check for Bingo
  const checkBingo = () => {
    bingoLines.value = []; // Reset lines before checking
    const rows = Array(size).fill(0).map((_, i) => boardFlat.value.slice(i * size, (i + 1) * size));
    const cols = Array(size).fill(0).map((_, i) => rows.map(row => row[i]));
    const diag1 = Array(size).fill(0).map((_, i) => rows[i][i]);
    const diag2 = Array(size).fill(0).map((_, i) => rows[i][size - i - 1]);
  
    const checkLine = (line) => line.every(cell => cell.selected);
  
    // Check rows, columns, and diagonals, adding each winning line to bingoLines
    rows.forEach((line, i) => {
      if (checkLine(line)) {
        line.forEach(cell => (cell.isBingo = true));
        bingoLines.value.push(calculateLineStyle('row', i));
      }
    });
    cols.forEach((line, i) => {
      if (checkLine(line)) {
        line.forEach(cell => (cell.isBingo = true));
        bingoLines.value.push(calculateLineStyle('col', i));
      }
    });
    if (checkLine(diag1)) {
      diag1.forEach(cell => (cell.isBingo = true));
      bingoLines.value.push(calculateLineStyle('diag1'));
    }
    if (checkLine(diag2)) {
      diag2.forEach(cell => (cell.isBingo = true));
      bingoLines.value.push(calculateLineStyle('diag2'));
    }
  
    if (bingoLines.value.length > 0) {
      bingo.value = true;
    }
  };
  
  // Calculate the winning line style dynamically
  const calculateLineStyle = (type, index = 0) => {
    const cellSize = 49; // Each cell's width/height
    const gap = 11; // Gap between cells
    const padding = 27; // Board padding
  
    if (type === 'row') {
      const topPosition = padding + index * (cellSize + gap) + cellSize / 2 - 1.5;
      return {
        top: `${topPosition}px`,
        left: `${padding}px`,
        width: `${size * (cellSize + gap) - gap}px`, // Width covers 5 cells minus gaps
        height: '3px',
        position: 'absolute',
        backgroundColor: 'rgba(82, 33, 156, 1)',
      };
    } else if (type === 'col') {
      const leftPosition = padding + index * (cellSize + gap) + cellSize / 2 - 1.5;
      return {
        top: `${padding}px`,
        left: `${leftPosition}px`,
        height: `${size * (cellSize + gap) - gap}px`, // Height covers 5 cells minus gaps
        width: '3px',
        position: 'absolute',
        backgroundColor: 'rgba(82, 33, 156, 1)',
      };
    } else if (type === 'diag1') {
      return {
        top: `${padding}px`,
        left: `${padding}px`,
        width: `${Math.sqrt(2) * (size * cellSize + (size - 1) * gap)}px`,
        height: '3px',
        position: 'absolute',
        backgroundColor: 'rgba(82, 33, 156, 1)',
        transform: 'rotate(45deg)',
        transformOrigin: '0 0',
      };
    } else if (type === 'diag2') {
      return {
        top: `${padding}px`,
        right: `${padding}px`,
        width: `${Math.sqrt(2) * (size * cellSize + (size - 1) * gap)}px`,
        height: '3px',
        position: 'absolute',
        backgroundColor: 'rgba(82, 33, 156, 1)',
        transform: 'rotate(-45deg)',
        transformOrigin: '100% 0',
      };
    }
  };
  
  // Reset the game
  const resetGame = () => {
    bingo.value = false;
    bingoLines.value = [];
    initBoard(); // Reinitialize the board
  };
  
  // Initialize the Bingo board on mount
  onMounted(() => {
    initBoard();
  });
  </script>
  
  <style scoped>
  .bingo-container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
  .bingo-board {
    position: relative;
    width: 343px;
    height: 342px;
    padding: 27px;
    display: grid;
    grid-template-columns: repeat(5, 49px);
    grid-template-rows: repeat(5, 49px);
    grid-gap: 11px;
    margin-top: 20px;
    box-sizing: border-box;
    background-color: white;
  }
  
  .bingo-cell {
    width: 49px;
    height: 49px;
    background-color: lightgray;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
  }
  
  .bingo-cell.selected {
    color: white;
  }
  
  .bingo-line {
    position: absolute;
    background-color: rgba(82, 33, 156, 1);
    z-index: 1;
  }
  
  button {
    margin-top: 20px;
  }
  </style>
  
