<script lang="ts">
    import { onMount, onDestroy, tick } from 'svelte';
  
    export let onGameOver = () => {};
  
    let snake = [{ x: 5, y: 5 }];
    let food = { x: 10, y: 5 };
    let direction = 'RIGHT';
    let nextDirection = 'RIGHT';
    let interval;
    let score = 0;
    let gameOver = false;
    const gridWidth = 20;
    const gridHeight = 10;
    
    function placeFood() {
      // Generate new food position that's not on the snake
      let newFood;
      do {
        newFood = {
          x: Math.floor(Math.random() * gridWidth),
          y: Math.floor(Math.random() * gridHeight)
        };
      } while (snake.some(s => s.x === newFood.x && s.y === newFood.y));
      
      food = newFood;
    }
  
    function move() {
      if (gameOver) return;
      
      // Update direction from nextDirection
      direction = nextDirection;
      
      const head = { ...snake[0] };
      
      // Move head according to direction
      if (direction === 'UP') head.y -= 1;
      if (direction === 'DOWN') head.y += 1;
      if (direction === 'LEFT') head.x -= 1;
      if (direction === 'RIGHT') head.x += 1;
  
      // Check for collisions with walls or self
      if (
        head.x < 0 || head.y < 0 || head.x >= gridWidth || head.y >= gridHeight ||
        snake.slice(1).some(s => s.x === head.x && s.y === head.y)
      ) {
        gameOver = true;
        clearInterval(interval);
        onGameOver();
        return;
      }
  
      // Add new head to snake
      snake = [head, ...snake];
      
      // Check if snake ate food
      if (head.x === food.x && head.y === food.y) {
        score += 10;
        placeFood();
      } else {
        // Remove tail if no food was eaten
        snake = snake.slice(0, -1);
      }
      
      // Force update
      snake = [...snake];
    }
  
    function handleKey(e: KeyboardEvent) {
      // Only update direction if it's not a 180-degree turn
      if ((e.key === 'ArrowUp' || e.key === 'w') && direction !== 'DOWN') {
        nextDirection = 'UP';
      } else if ((e.key === 'ArrowDown' || e.key === 's') && direction !== 'UP') {
        nextDirection = 'DOWN';
      } else if ((e.key === 'ArrowLeft' || e.key === 'a') && direction !== 'RIGHT') {
        nextDirection = 'LEFT';
      } else if ((e.key === 'ArrowRight' || e.key === 'd') && direction !== 'LEFT') {
        nextDirection = 'RIGHT';
      }
    }
  
    onMount(async () => {
      // Need to wait a bit to ensure the component is fully mounted
      await tick();
      window.addEventListener('keydown', handleKey);
      interval = setInterval(move, 150);
      
      // Initialize the game
      placeFood();
    });
  
    onDestroy(() => {
      clearInterval(interval);
      window.removeEventListener('keydown', handleKey);
    });
</script>
  
<div class="snake-game">
  <div class="game-info text-green-400 mb-2">
    <span>Score: {score}</span>
    {#if gameOver}
      <span class="ml-4 text-red-500">GAME OVER! Press CTRL+C to exit.</span>
    {/if}
  </div>
  
  <div class="game-board">
    <table class="border-collapse">
      <tbody>
        {#each Array(gridHeight) as _, y}
          <tr>
            {#each Array(gridWidth) as _, x}
              <td style="padding: 0; width: 16px; height: 16px; text-align: center; line-height: 1;">
                {#if snake.some(s => s.x === x && s.y === y)}
                  <div class="inline-block w-4 h-4 bg-green-500" style="border-radius: 2px;"></div>
                {:else if food.x === x && food.y === y}
                  <div class="inline-block w-4 h-4 bg-red-500" style="border-radius: 50%;"></div>
                {:else}
                  <div class="inline-block w-4 h-4 bg-{x === 0 || x === gridWidth-1 || y === 0 || y === gridHeight-1 ? "gray-800" : "transparent"}"></div>
                {/if}
              </td>
            {/each}
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
  
  <div class="mt-2 text-green-400 text-xs">
    <p>Controls: Arrow keys or WASD to move. CTRL+C to exit.</p>
  </div>
</div>

<style>
  .snake-game {
    padding: 10px;
  }
</style>