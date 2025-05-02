<!-- <script lang="ts">
    import { onMount, onDestroy, tick } from 'svelte';
  
    export let onGameOver = () => {};
  
    const GRID_WIDTH = 10;
    const GRID_HEIGHT = 16;
    let grid = Array(GRID_HEIGHT).fill(null).map(() => Array(GRID_WIDTH).fill(0));
    let score = 0;
    let gameOver = false;
    let interval;
    let currentPiece = null;
    let currentPosition = { x: 0, y: 0 };
  
    // Tetris pieces and their rotations
    const PIECES = [
      // I piece - cyan
      {
        shapes: [
          [[1, 1, 1, 1]],
          [[1], [1], [1], [1]]
        ],
        color: 1
      },
      // O piece - yellow
      {
        shapes: [
          [[2, 2], [2, 2]]
        ],
        color: 2
      },
      // T piece - purple
      {
        shapes: [
          [[0, 3, 0], [3, 3, 3]],
          [[3, 0], [3, 3], [3, 0]],
          [[3, 3, 3], [0, 3, 0]],
          [[0, 3], [3, 3], [0, 3]]
        ],
        color: 3
      },
      // J piece - blue
      {
        shapes: [
          [[4, 0, 0], [4, 4, 4]],
          [[4, 4], [4, 0], [4, 0]],
          [[4, 4, 4], [0, 0, 4]],
          [[0, 4], [0, 4], [4, 4]]
        ],
        color: 4
      },
      // L piece - orange
      {
        shapes: [
          [[0, 0, 5], [5, 5, 5]],
          [[5, 0], [5, 0], [5, 5]],
          [[5, 5, 5], [5, 0, 0]],
          [[5, 5], [0, 5], [0, 5]]
        ],
        color: 5
      },
      // S piece - green
      {
        shapes: [
          [[0, 6, 6], [6, 6, 0]],
          [[6, 0], [6, 6], [0, 6]]
        ],
        color: 6
      },
      // Z piece - red
      {
        shapes: [
          [[7, 7, 0], [0, 7, 7]],
          [[0, 7], [7, 7], [7, 0]]
        ],
        color: 7
      }
    ];
  
    // Color mapping
    const COLORS = {
      0: '⬛', // empty
      1: '🟦', // cyan
      2: '🟨', // yellow
      3: '🟪', // purple
      4: '🟦', // blue
      5: '🟧', // orange
      6: '🟩', // green
      7: '🟥'  // red
    };
  
    // Current piece state
    let currentRotation = 0;
  
    function getRandomPiece() {
      const piece = PIECES[Math.floor(Math.random() * PIECES.length)];
      currentRotation = 0;
      return piece;
    }
  
    function spawnPiece() {
      currentPiece = getRandomPiece();
      const pieceWidth = currentPiece.shapes[currentRotation][0].length;
      currentPosition = {
        x: Math.floor((GRID_WIDTH - pieceWidth) / 2),
        y: 0
      };
  
      // Check if the piece can be placed at the starting position
      if (!isValidMove(currentPosition.x, currentPosition.y, currentRotation)) {
        gameOver = true;
        clearInterval(interval);
        onGameOver();
      }
    }
  
    function isValidMove(x, y, rotation) {
      const shape = currentPiece.shapes[rotation];
      for (let row = 0; row < shape.length; row++) {
        for (let col = 0; col < shape[row].length; col++) {
          if (shape[row][col] !== 0) {
            const nextX = x + col;
            const nextY = y + row;
  
            // Check boundaries
            if (nextX < 0 || nextX >= GRID_WIDTH || nextY >= GRID_HEIGHT) {
              return false;
            }
  
            // Check collision with placed pieces (but not out of bounds)
            if (nextY >= 0 && grid[nextY][nextX] !== 0) {
              return false;
            }
          }
        }
      }
      return true;
    }
  
    function mergePieceToGrid() {
      const shape = currentPiece.shapes[currentRotation];
      for (let row = 0; row < shape.length; row++) {
        for (let col = 0; col < shape[row].length; col++) {
          if (shape[row][col] !== 0) {
            const gridY = currentPosition.y + row;
            const gridX = currentPosition.x + col;
            if (gridY >= 0) { // Only merge if it's in bounds
              grid[gridY][gridX] = currentPiece.color;
            }
          }
        }
      }
      
      // Check for completed lines
      checkLines();
      
      // Spawn a new piece
      spawnPiece();
    }
  
    function moveDown() {
      if (gameOver) return;
      
      if (isValidMove(currentPosition.x, currentPosition.y + 1, currentRotation)) {
        currentPosition.y++;
      } else {
        mergePieceToGrid();
      }
    }
  
    function moveLeft() {
      if (isValidMove(currentPosition.x - 1, currentPosition.y, currentRotation)) {
        currentPosition.x--;
      }
    }
  
    function moveRight() {
      if (isValidMove(currentPosition.x + 1, currentPosition.y, currentRotation)) {
        currentPosition.x++;
      }
    }
  
    function rotate() {
      const nextRotation = (currentRotation + 1) % currentPiece.shapes.length;
      if (isValidMove(currentPosition.x, currentPosition.y, nextRotation)) {
        currentRotation = nextRotation;
      }
    }
  
    function hardDrop() {
      while (isValidMove(currentPosition.x, currentPosition.y + 1, currentRotation)) {
        currentPosition.y++;
      }
      mergePieceToGrid();
    }
  
    function checkLines() {
      let completedLines = 0;
      
      for (let row = GRID_HEIGHT - 1; row >= 0; row--) {
        if (grid[row].every(cell => cell !== 0)) {
          // Remove the line
          grid.splice(row, 1);
          // Add a new empty line at the top
          grid.unshift(Array(GRID_WIDTH).fill(0));
          completedLines++;
          
          // Since we removed a line, we need to check the same row again
          row++;
        }
      }
      
      // Update score based on completed lines
      if (completedLines > 0) {
        const points = [0, 40, 100, 300, 1200][completedLines];
        score += points;
      }
    }
  
    function getDisplayGrid() {
      // Create a copy of the grid
      const displayGrid = grid.map(row => [...row]);
      
      // Add the current piece to the display grid
      if (currentPiece) {
        const shape = currentPiece.shapes[currentRotation];
        for (let row = 0; row < shape.length; row++) {
          for (let col = 0; col < shape[row].length; col++) {
            if (shape[row][col] !== 0) {
              const gridY = currentPosition.y + row;
              const gridX = currentPosition.x + col;
              if (gridY >= 0 && gridY < GRID_HEIGHT && gridX >= 0 && gridX < GRID_WIDTH) {
                displayGrid[gridY][gridX] = currentPiece.color;
              }
            }
          }
        }
      }
      
      return displayGrid;
    }
  
    function handleKey(e: KeyboardEvent) {
      if (gameOver) return;
      
      if (e.key === 'ArrowLeft' || e.key === 'a') {
        moveLeft();
      } else if (e.key === 'ArrowRight' || e.key === 'd') {
        moveRight();
      } else if (e.key === 'ArrowDown' || e.key === 's') {
        moveDown();
      } else if (e.key === 'ArrowUp' || e.key === 'w') {
        rotate();
      } else if (e.key === ' ') {
        hardDrop();
      }
    }
  
    function gameLoop() {
      moveDown();
    }
  
    onMount(async () => {
      await tick();
      window.addEventListener('keydown', handleKey);
      spawnPiece();
      interval = setInterval(gameLoop, 500);
    });
  
    onDestroy(() => {
      clearInterval(interval);
      window.removeEventListener('keydown', handleKey);
    });
  
    $: displayGrid = getDisplayGrid();
  </script>
  
  <div class="tetris-game">
    <div class="game-info text-green-400 mb-2">
      <span>Score: {score}</span>
      {#if gameOver}
        <span class="ml-4 text-red-500">GAME OVER! Press CTRL+C to exit.</span>
      {/if}
    </div>
    
    <pre class="leading-none font-mono text-green-400 text-xs">
      {#each displayGrid as row}
        {#each row as cell}
          {COLORS[cell]}
        {/each}
        {'\n'}
      {/each}
    </pre>
    
    <div class="mt-2 text-green-400 text-xs">
      <p>Controls: Arrow keys or WASD to move, Space to hard drop. CTRL+C to exit.</p>
    </div>
  </div> -->

  <div class="text-green-400 text-xs"> Still needs some work, CTRL+C to exit</div>