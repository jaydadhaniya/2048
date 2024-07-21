<template>
  <div id="app" class="dF fC aC">
    <div class="dF aC jSB" style="width: 500px;">
      <div>
        <div class="title">2048</div>
        <div class="description"> Join the tiles, get to <strong>2048!</strong></div>
      </div>
      <div class="dF fC aE jSB" style="gap: 20px;">
        <div class="dF aC jSB" style="gap: 20px;">
          <div class="score-container">
            <div class="score-title">SCORE</div>
            <div class="score-value">{{ score }} </div>
          </div>
          <div class="score-container">
            <div class="score-title">BEST</div>
            <div class="score-value"> {{ bestScore }} </div>
          </div>
        </div>
        <button class="btn new-game" @click="startNewGame"> NEW GAME</button>
      </div>
    </div>
    <div class="grid-container">
      <transition-group name="tile" tag="div">
        <div class="grid-row" v-for="(row, rowIndex) in grid" :key="rowIndex">
          <div class="grid-cell" v-for="(col, colIndex) in row" :key="colIndex">
            <div class="tile"
              :class="`${grid[rowIndex][colIndex] ? `tile-${grid[rowIndex][colIndex]}` : ''} ${gridClass[rowIndex][colIndex]}`">
              <div class="tile-inner">
                {{ grid[rowIndex][colIndex] || '' }}
              </div>
            </div>
          </div>
        </div>
      </transition-group>
      <div v-if="isGameOver || isGameWon" class="game-message game-over">
        <p v-if="isGameWon">You Win!</p>
        <p v-else>Game over!</p>
        <div class="dF aC" style="gap: 25px; margin-top: 25px;">
          <button v-if="isGameWon" class="btn" @click="keepGoing"><strong>Keep going</strong></button>
          <button class="btn" @click="tryAgain"><strong>Try again</strong></button>
        </div>
      </div>
    </div>
    <div class="explanation">
      <strong> HOW TO PLAY:</strong> Use your <strong>arrow keys</strong> to move the tiles. Tiles with the same number
      <strong>merge into one</strong>
      when they touch. Add
      them up to reach <strong>2048!</strong>
    </div>
  </div>
</template>

<script setup>

  import { onMounted, onUnmounted, ref } from 'vue';

  // Reactive variables to hold grid length and default states
  const gridLength = ref(4); // Default grid length (4x4)

  // Initialize default grid and class for styling
  const defaultGrid = Array.from({ length: gridLength.value }, () => Array.from({ length: gridLength.value }, () => 0));
  const defaultGridClass = Array.from({ length: gridLength.value }, () => Array.from({ length: gridLength.value }, () => ''));

  // Retrieve saved game state from localStorage
  const saveGrid = localStorage.getItem('grid');
  const saveScore = localStorage.getItem('score');
  const saveBestScore = localStorage.getItem('bestScore');

  // Reactive variables for the current game state
  const grid = ref(saveGrid ? JSON.parse(saveGrid) : JSON.parse(JSON.stringify(defaultGrid)));
  const gridClass = ref(JSON.parse(JSON.stringify(defaultGridClass)));
  const isGameOver = ref(false); // Indicates if the game is over
  const isGameWon = ref(false); // Indicates if the player has won
  const winNumber = ref(2048); // Number required to win the game
  const score = ref(saveScore ? Number(saveScore) : 0); // Current score
  const bestScore = ref(saveBestScore ? Number(saveBestScore) : 0); // Best score

  score.value = 12345378;
  bestScore.value = 12345678;

  // Lifecycle hook to initialize the game and add event listeners
  onMounted(() => {
    if (!saveGrid) {
      addRandomTile();
      addRandomTile();
    }
    window.addEventListener('keydown', handleKeyDown); // Add event listener for keyboard input
  });

  // Cleanup event listener on component unmount
  onUnmounted(() => {
    window.removeEventListener('keydown', handleKeyDown); // Remove event listener
  });

  // Function to handle key presses for moving tiles
  const handleKeyDown = (event) => {
    event.preventDefault();
    gridClass.value = JSON.parse(JSON.stringify(defaultGridClass)); // Reset tile classes
    const previousGrid = JSON.parse(JSON.stringify(grid.value)); // Save current grid state for comparison

    let validEvent = false;
    switch (event.key) {
      case 'ArrowUp':
        validEvent = true;
        moveUp(); // Move tiles up
        break;
      case 'ArrowDown':
        validEvent = true;
        moveDown(); // Move tiles down
        break;
      case 'ArrowLeft':
        validEvent = true;
        moveLeft(); // Move tiles left
        break;
      case 'ArrowRight':
        validEvent = true;
        moveRight(); // Move tiles right
        break;
    }

    // Add a new tile and check game status if a valid move was made
    if (validEvent && gridsAreDifferent(previousGrid, grid.value)) {
      addRandomTile();
      if (checkGameOver()) {
        isGameOver.value = true; // Set game over status
      }
      checkGameWon(); // Check if the game is won
      saveDataInLocalStorage(); // Save game state
    }
  };

  // Functions to handle movement in each direction
  const moveUp = () => {
    for (let colIndex = 0; colIndex < gridLength.value; colIndex++) {
      let writeIndex = 0; // Position where the next non-zero value should be written
      let previousValue = null; // Track the previous value for merging

      for (let rowIndex = 0; rowIndex < gridLength.value; rowIndex++) {
        const value = grid.value[rowIndex][colIndex];

        if (value) {
          if (previousValue === value) {
            // Merge with the previous value
            grid.value[writeIndex - 1][colIndex] *= 2;
            gridClass.value[writeIndex - 1][colIndex] = 'tile-merged';
            previousValue = null; // Reset previousValue after merge
            updateScore(value * 2);
          } else {
            // Write current value to the correct position
            grid.value[writeIndex][colIndex] = value;
            writeIndex++;
            previousValue = value;
          }
        }
      }

      // Fill the remaining positions in the column with zeros
      for (let rowIndex = writeIndex; rowIndex < gridLength.value; rowIndex++) {
        grid.value[rowIndex][colIndex] = 0;
      }
    }
  };

  const moveDown = () => {
    for (let colIndex = 0; colIndex < gridLength.value; colIndex++) {
      let writeIndex = gridLength.value - 1; // Position where the next non-zero value should be written
      let previousValue = null; // Track the previous value for merging

      for (let rowIndex = gridLength.value - 1; rowIndex >= 0; rowIndex--) {
        const value = grid.value[rowIndex][colIndex];

        if (value) {
          if (previousValue === value) {
            // Merge with the previous value
            grid.value[writeIndex + 1][colIndex] *= 2;
            gridClass.value[writeIndex + 1][colIndex] = 'tile-merged';
            previousValue = null; // Reset previousValue after merge
            updateScore(value * 2);
          } else {
            // Write current value to the correct position
            grid.value[writeIndex][colIndex] = value;
            writeIndex--;
            previousValue = value;
          }
        }
      }

      // Fill the remaining positions in the column with zeros
      for (let rowIndex = writeIndex; rowIndex >= 0; rowIndex--) {
        grid.value[rowIndex][colIndex] = 0;
      }
    }
  };

  const moveLeft = () => {
    for (let rowIndex = 0; rowIndex < gridLength.value; rowIndex++) {
      let writeIndex = 0; // Position where the next non-zero value should be written
      let previousValue = null; // Track the previous value for merging

      for (let colIndex = 0; colIndex < gridLength.value; colIndex++) {
        const value = grid.value[rowIndex][colIndex];

        if (value) {
          if (previousValue === value) {
            // Merge with the previous value
            grid.value[rowIndex][writeIndex - 1] *= 2;
            gridClass.value[rowIndex][writeIndex - 1] = 'tile-merged';
            previousValue = null; // Reset previousValue after merge
            updateScore(value * 2);
          } else {
            // Write current value to the correct position
            grid.value[rowIndex][writeIndex] = value;
            writeIndex++;
            previousValue = value;
          }
        }
      }

      // Fill the remaining positions in the row with zeros
      for (let colIndex = writeIndex; colIndex < gridLength.value; colIndex++) {
        grid.value[rowIndex][colIndex] = 0;
      }
    }
  };

  const moveRight = () => {
    for (let rowIndex = 0; rowIndex < gridLength.value; rowIndex++) {
      let writeIndex = gridLength.value - 1; // Position where the next non-zero value should be written
      let previousValue = null; // Track the previous value for merging

      for (let colIndex = gridLength.value - 1; colIndex >= 0; colIndex--) {
        const value = grid.value[rowIndex][colIndex];

        if (value) {
          if (previousValue === value) {
            // Merge with the previous value
            grid.value[rowIndex][writeIndex + 1] *= 2;
            gridClass.value[rowIndex][writeIndex + 1] = 'tile-merged';
            previousValue = null; // Reset previousValue after merge
            updateScore(value * 2);
          } else {
            // Write current value to the correct position
            grid.value[rowIndex][writeIndex] = value;
            writeIndex--;
            previousValue = value;
          }
        }
      }

      // Fill the remaining positions in the row with zeros
      for (let colIndex = writeIndex; colIndex >= 0; colIndex--) {
        grid.value[rowIndex][colIndex] = 0;
      }
    }
  };

  // Function to start a new game
  const startNewGame = () => {
    if (confirm('Are you sure you want to start a new game? All progress will be lost.')) {
      tryAgain(); // Reset the game
    }
  };

  // Function to reset the game state and start a new game
  const tryAgain = () => {
    grid.value = JSON.parse(JSON.stringify(defaultGrid));
    gridClass.value = JSON.parse(JSON.stringify(defaultGridClass));
    // Add two starting tiles
    addRandomTile();
    addRandomTile();

    isGameOver.value = false;
    isGameWon.value = false;
    score.value = 0;

    // Clear saved state
    localStorage.removeItem('grid');
    localStorage.removeItem('score');
  };

  // Function to continue the game after reaching the winning number
  const keepGoing = () => {
    // Double the winning number for next goal
    winNumber.value *= 2;
    isGameWon.value = false;
    isGameOver.value = false;
  };

  // Function to update the score and best score
  const updateScore = (value) => {
    score.value += value;
    // Update best score if current score is higher
    if (score.value > bestScore.value) {
      bestScore.value = score.value;
    }
  };

  // Function to check if the game is over (no more moves possible)
  const checkGameOver = () => {
    // Check if any empty tiles are available
    for (let row of grid.value) {
      if (row.includes(0)) return false;
    }

    // Check if there are any adjacent tiles with the same value
    for (let row = 0; row < gridLength.value; row++) {
      for (let col = 0; col < gridLength.value; col++) {
        if (col < gridLength.value - 1 && grid.value[row][col] === grid.value[row][col + 1]) return false;
        if (row < gridLength.value - 1 && grid.value[row][col] === grid.value[row + 1][col]) return false;
      }
    }

    return true; // No moves left
  };

  // Function to check if the player has won (reached the win number)
  const checkGameWon = () => {
    for (let row of grid.value) {
      if (row.includes(winNumber.value)) {
        isGameWon.value = true;
        return true;
      }
    }
    return false;
  };

  // Function to compare two grids and determine if they are different
  const gridsAreDifferent = (grid1, grid2) => {
    for (let rowIndex = 0; rowIndex < grid1.length; rowIndex++) {
      for (let colIndex = 0; colIndex < grid1[rowIndex].length; colIndex++) {
        if (grid1[rowIndex][colIndex] !== grid2[rowIndex][colIndex]) {
          return true; // Grids are different
        }
      }
    }
    return false; // Grids are the same
  };

  // Function to add a random tile (2 or 4) to an empty position
  const addRandomTile = () => {
    const emptyTiles = [];
    grid.value.forEach((row, rowIndex) => {
      row.forEach((tile, colIndex) => {
        if (tile === 0) {
          emptyTiles.push({ rowIndex, colIndex });
        }
      });
    });

    if (emptyTiles.length > 0) {
      const { rowIndex, colIndex } = emptyTiles[Math.floor(Math.random() * emptyTiles.length)];
      grid.value[rowIndex][colIndex] = Math.random() < 0.99 ? 2 : 4; // Randomly place a 2 or 4
      gridClass.value[rowIndex][colIndex] = 'tile-new'; // Add new tile styling
    }
  };

  // Function to save game state in localStorage
  const saveDataInLocalStorage = () => {
    localStorage.setItem('grid', JSON.stringify(grid.value));
    localStorage.setItem('score', score.value);
    localStorage.setItem('bestScore', bestScore.value);
  };

</script>

<style scoped>
  .title {
    font-size: 80px;
    font-weight: bold;
  }

  .description {
    font-size: 1rem;
  }

  .score-container {
    text-align: center;
    border: 1px solid;
    padding: 2px 10px;
    background-color: #bbada0;
    color: white;
    border: none;
    border-radius: 2px;
  }

  .score-title {
    color: #eee4da;
    font-size: 15px;
    font-weight: bold;
  }

  .score-value {
    font-size: 20px;
    font-weight: bold;
  }

  .btn {
    cursor: pointer;
    font-size: medium;
    padding: 10px 20px;
    background-color: #8F7A66;
    color: white;
    border: none;
    border-radius: 2px;
    font-weight: bold;
  }

  .explanation {
    margin-top: 2rem;
    max-width: 500px;
    font-size: large;
  }

  .grid-container {
    position: relative;
    margin-top: 2rem;
    cursor: default;
    user-select: none;
    touch-action: none;
    width: 500px;
    height: 500px;
    box-sizing: border-box;
    padding: 15px;
    background: rgb(187, 173, 160);
    border-radius: 6px;
  }

  .grid-row {
    margin-bottom: 15px;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .grid-cell {
    width: 100px;
    height: 100px;
    border-radius: 3px;
    background: rgba(238, 228, 218, 0.35);
  }

  .tile {
    width: 100px;
    height: 100px;
    position: absolute;
    -webkit-transition: 100ms ease-in-out;
    -moz-transition: 100ms ease-in-out;
    transition: 100ms ease-in-out;
    -webkit-transition-property: -webkit-transform;
    -moz-transition-property: -moz-transform;
    transition-property: transform;
  }

  .tile .tile-inner {
    line-height: 100px;
    border-radius: 3px;
    background: #eee4da;
    display: flex;
    justify-content: center;
    text-align: center;
    font-weight: bold;
    z-index: 10;
    font-size: 55px;
  }

  .tile-new .tile-inner {
    -webkit-animation: appear 200ms ease 100ms;
    -moz-animation: appear 200ms ease 100ms;
    animation: appear 200ms ease 100ms;
    -webkit-animation-fill-mode: backwards;
    -moz-animation-fill-mode: backwards;
    animation-fill-mode: backwards;
  }

  .tile-merged .tile-inner {
    z-index: 20;
    -webkit-animation: pop 200ms ease 100ms;
    -moz-animation: pop 200ms ease 100ms;
    animation: pop 200ms ease 100ms;
    -webkit-animation-fill-mode: backwards;
    -moz-animation-fill-mode: backwards;
    animation-fill-mode: backwards;
  }

  .tile.tile-2 .tile-inner {
    background: #eee4da;
    box-shadow: 0 0 30px 10px rgba(243, 215, 116, 0), inset 0 0 0 1px rgba(255, 255, 255, 0);
  }

  .tile.tile-4 .tile-inner {
    background: #eee1c9;
    box-shadow: 0 0 30px 10px rgba(243, 215, 116, 0), inset 0 0 0 1px rgba(255, 255, 255, 0);
  }

  .tile.tile-8 .tile-inner {
    color: #f9f6f2;
    background: #f3b27a;
  }

  .tile.tile-16 .tile-inner {
    color: #f9f6f2;
    background: #f69664;
  }

  .tile.tile-32 .tile-inner {
    color: #f9f6f2;
    background: #f77c5f;
  }

  .tile.tile-64 .tile-inner {
    color: #f9f6f2;
    background: #f75f3b;
  }

  .tile.tile-128 .tile-inner {
    color: #f9f6f2;
    background: #edd073;
    box-shadow: 0 0 30px 10px rgba(243, 215, 116, 0.2380952381), inset 0 0 0 1px rgba(255, 255, 255, 0.1428571429);
    font-size: 45px;
  }

  @media screen and (max-width: 520px) {
    .tile.tile-128 .tile-inner {
      font-size: 25px;
    }
  }

  .tile.tile-256 .tile-inner {
    color: #f9f6f2;
    background: #edcc62;
    box-shadow: 0 0 30px 10px rgba(243, 215, 116, 0.3174603175), inset 0 0 0 1px rgba(255, 255, 255, 0.1904761905);
    font-size: 45px;
  }

  @media screen and (max-width: 520px) {
    .tile.tile-256 .tile-inner {
      font-size: 25px;
    }
  }

  .tile.tile-512 .tile-inner {
    color: #f9f6f2;
    background: #edc950;
    box-shadow: 0 0 30px 10px rgba(243, 215, 116, 0.3968253968), inset 0 0 0 1px rgba(255, 255, 255, 0.2380952381);
    font-size: 45px;
  }

  @media screen and (max-width: 520px) {
    .tile.tile-512 .tile-inner {
      font-size: 25px;
    }
  }

  .tile.tile-1024 .tile-inner {
    color: #f9f6f2;
    background: #edc53f;
    box-shadow: 0 0 30px 10px rgba(243, 215, 116, 0.4761904762), inset 0 0 0 1px rgba(255, 255, 255, 0.2857142857);
    font-size: 35px;
  }

  @media screen and (max-width: 520px) {
    .tile.tile-1024 .tile-inner {
      font-size: 15px;
    }
  }

  .tile.tile-2048 .tile-inner {
    color: #f9f6f2;
    background: #edc22e;
    box-shadow: 0 0 30px 10px rgba(243, 215, 116, 0.5555555556), inset 0 0 0 1px rgba(255, 255, 255, 0.3333333333);
    font-size: 35px;
  }

  @media screen and (max-width: 520px) {
    .tile.tile-2048 .tile-inner {
      font-size: 15px;
    }
  }

  .tile.tile-super .tile-inner {
    color: #f9f6f2;
    background: #3c3a33;
    font-size: 30px;
  }

  @media screen and (max-width: 520px) {
    .tile.tile-super .tile-inner {
      font-size: 10px;
    }
  }

  .game-message {
    display: none;
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background: rgba(238, 228, 218, 0.73);
    z-index: 100;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    -webkit-animation: fade-in 800ms ease 1200ms;
    -moz-animation: fade-in 800ms ease 1200ms;
    animation: fade-in 800ms ease 1200ms;
    -webkit-animation-fill-mode: both;
    -moz-animation-fill-mode: both;
    animation-fill-mode: both;
  }

  .game-message p {
    font-size: 60px;
    font-weight: bold;
    height: 60px;
    line-height: 60px;
  }

  .game-message .lower {
    display: block;
    margin-top: 29px;
  }

  .game-message a {
    display: inline-block;
    background: #8f7a66;
    border-radius: 3px;
    padding: 0 20px;
    text-decoration: none;
    color: #f9f6f2;
    height: 40px;
    line-height: 42px;
    cursor: pointer;
    margin-left: 9px;
  }

  .game-message a.keep-playing-button {
    display: none;
  }

  .game-message.game-won {
    background: rgba(237, 194, 46, 0.5);
    color: #f9f6f2;
  }

  .game-message.game-won a.keep-playing-button {
    display: inline-block;
  }

  .game-message.game-won,
  .game-message.game-over {
    display: flex;
  }


  @-webkit-keyframes appear {
    0% {
      opacity: 0;
      -webkit-transform: scale(0);
      -moz-transform: scale(0);
      -ms-transform: scale(0);
      transform: scale(0);
    }

    100% {
      opacity: 1;
      -webkit-transform: scale(1);
      -moz-transform: scale(1);
      -ms-transform: scale(1);
      transform: scale(1);
    }
  }

  @-moz-keyframes appear {
    0% {
      opacity: 0;
      -webkit-transform: scale(0);
      -moz-transform: scale(0);
      -ms-transform: scale(0);
      transform: scale(0);
    }

    100% {
      opacity: 1;
      -webkit-transform: scale(1);
      -moz-transform: scale(1);
      -ms-transform: scale(1);
      transform: scale(1);
    }
  }

  @keyframes appear {
    0% {
      opacity: 0;
      -webkit-transform: scale(0);
      -moz-transform: scale(0);
      -ms-transform: scale(0);
      transform: scale(0);
    }

    100% {
      opacity: 1;
      -webkit-transform: scale(1);
      -moz-transform: scale(1);
      -ms-transform: scale(1);
      transform: scale(1);
    }
  }


  @-webkit-keyframes pop {
    0% {
      -webkit-transform: scale(0);
      -moz-transform: scale(0);
      -ms-transform: scale(0);
      transform: scale(0);
    }

    50% {
      -webkit-transform: scale(1.2);
      -moz-transform: scale(1.2);
      -ms-transform: scale(1.2);
      transform: scale(1.2);
    }

    100% {
      -webkit-transform: scale(1);
      -moz-transform: scale(1);
      -ms-transform: scale(1);
      transform: scale(1);
    }
  }

  @-moz-keyframes pop {
    0% {
      -webkit-transform: scale(0);
      -moz-transform: scale(0);
      -ms-transform: scale(0);
      transform: scale(0);
    }

    50% {
      -webkit-transform: scale(1.2);
      -moz-transform: scale(1.2);
      -ms-transform: scale(1.2);
      transform: scale(1.2);
    }

    100% {
      -webkit-transform: scale(1);
      -moz-transform: scale(1);
      -ms-transform: scale(1);
      transform: scale(1);
    }
  }

  @keyframes pop {
    0% {
      -webkit-transform: scale(0);
      -moz-transform: scale(0);
      -ms-transform: scale(0);
      transform: scale(0);
    }

    50% {
      -webkit-transform: scale(1.2);
      -moz-transform: scale(1.2);
      -ms-transform: scale(1.2);
      transform: scale(1.2);
    }

    100% {
      -webkit-transform: scale(1);
      -moz-transform: scale(1);
      -ms-transform: scale(1);
      transform: scale(1);
    }
  }

  @-webkit-keyframes move-up {
    0% {
      top: 25px;
      opacity: 1;
    }

    100% {
      top: -50px;
      opacity: 0;
    }
  }

  @-moz-keyframes move-up {
    0% {
      top: 25px;
      opacity: 1;
    }

    100% {
      top: -50px;
      opacity: 0;
    }
  }

  @keyframes move-up {
    0% {
      top: 25px;
      opacity: 1;
    }

    100% {
      top: -50px;
      opacity: 0;
    }
  }

  @-webkit-keyframes fade-in {
    0% {
      opacity: 0;
    }

    100% {
      opacity: 1;
    }
  }

  @-moz-keyframes fade-in {
    0% {
      opacity: 0;
    }

    100% {
      opacity: 1;
    }
  }

  @keyframes fade-in {
    0% {
      opacity: 0;
    }

    100% {
      opacity: 1;
    }
  }
</style>
