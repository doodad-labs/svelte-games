<script lang="ts">
    import { browser } from '$app/environment';
    import { onMount, onDestroy } from 'svelte';

    // Upscale the game for better visibility
    const canvasSize = 300 * 2;
    const gridSize = 20 * 2;
    const gridCount = canvasSize / gridSize;

    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;
    let snake: { x: number; y: number }[] = [];
    let food = { x: 5, y: 5 };
    let direction = { x: 0, y: 0 };
    let gameInterval: number | null = null;
    let gameStarted = false;
    let baseSpeed = 200; // Base speed in milliseconds
    let speedMultiplier = 1; // Multiplier to adjust speed based on snake length
    let touchStartX = 0; // X coordinate of touch start
    let touchStartY = 0; // Y coordinate of touch start

    const drawRect = (x: number, y: number, color: string) => {
        ctx.fillStyle = color;
        ctx.fillRect(x * gridSize, y * gridSize, gridSize, gridSize);
    };

    const drawSnake = () => {
        snake.forEach(segment => drawRect(segment.x, segment.y, 'lime'));
    };

    const drawFood = () => {
        drawRect(food.x, food.y, 'red');
    };

    const drawGrid = () => {
        ctx.strokeStyle = 'rgba(0, 0, 0, 0.2)'; // Grid line color
        ctx.lineWidth = 1;

        // Draw vertical lines
        for (let x = 0; x <= canvasSize; x += gridSize) {
            ctx.beginPath();
            ctx.moveTo(x, 0);
            ctx.lineTo(x, canvasSize);
            ctx.stroke();
        }

        // Draw horizontal lines
        for (let y = 0; y <= canvasSize; y += gridSize) {
            ctx.beginPath();
            ctx.moveTo(0, y);
            ctx.lineTo(canvasSize, y);
            ctx.stroke();
        }
    };

    const moveSnake = () => {
        const head = { x: snake[0].x + direction.x, y: snake[0].y + direction.y };

        // Check for collision with walls
        if (head.x < 0 || head.x >= gridCount || head.y < 0 || head.y >= gridCount) {
            resetGame();
            return;
        }

        // Check for collision with itself
        if (snake.some(segment => segment.x === head.x && segment.y === head.y)) {
            resetGame();
            return;
        }

        snake.unshift(head);

        // Check if snake eats food
        if (head.x === food.x && head.y === food.y) {
            placeFood();
            // Increase speed as the snake grows
            speedMultiplier = Math.max(0.5, 1 - snake.length * 0.02); // Adjust multiplier based on snake length
            updateGameInterval();
        } else {
            snake.pop();
        }
    };

    const placeFood = () => {
        let newFoodPosition: { x: number; y: number };
        do {
            newFoodPosition = {
                x: Math.floor(Math.random() * gridCount),
                y: Math.floor(Math.random() * gridCount)
            };
        } while (snake.some(segment => segment.x === newFoodPosition.x && segment.y === newFoodPosition.y));

        food = newFoodPosition;
    };

    const getRandomPosition = () => {
        let position: { x: number; y: number };
        do {
            position = {
                x: Math.floor(Math.random() * gridCount),
                y: Math.floor(Math.random() * gridCount)
            };
        } while (snake.some(segment => segment.x === position.x && segment.y === position.y));

        return position;
    };

    const resetGame = () => {
        // Set the snake to a random position
        const randomPosition = getRandomPosition();
        snake = [{ x: randomPosition.x, y: randomPosition.y }];
        direction = { x: 0, y: 0 };
        gameStarted = false;
        speedMultiplier = 1; // Reset speed multiplier
        placeFood();
        if (gameInterval) {
            clearInterval(gameInterval);
            gameInterval = null;
        }
        drawGame(); // Redraw the initial state
    };

    const updateGameInterval = () => {
        if (gameInterval) {
            clearInterval(gameInterval);
        }
        const intervalDelay = baseSpeed * speedMultiplier; // Adjust delay based on speed multiplier
        gameInterval = setInterval(() => {
            moveSnake();
            drawGame();
        }, intervalDelay);
    };

    const handleKeyDown = (event: KeyboardEvent) => {
        const keys = ['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'];
        if (!keys.includes(event.key)) return;

        if (!gameStarted) {
            gameStarted = true;
            updateGameInterval(); // Start the game with the initial interval
        }

        switch (event.key) {
            case 'ArrowUp':
                if (direction.y === 0) direction = { x: 0, y: -1 };
                break;
            case 'ArrowDown':
                if (direction.y === 0) direction = { x: 0, y: 1 };
                break;
            case 'ArrowLeft':
                if (direction.x === 0) direction = { x: -1, y: 0 };
                break;
            case 'ArrowRight':
                if (direction.x === 0) direction = { x: 1, y: 0 };
                break;
        }
    };

    const handleTouchStart = (event: TouchEvent) => {
        touchStartX = event.touches[0].clientX;
        touchStartY = event.touches[0].clientY;
    };

    const handleTouchEnd = (event: TouchEvent) => {
        const touchEndX = event.changedTouches[0].clientX;
        const touchEndY = event.changedTouches[0].clientY;

        const deltaX = touchEndX - touchStartX;
        const deltaY = touchEndY - touchStartY;

        if (Math.abs(deltaX) > Math.abs(deltaY)) {
            // Horizontal swipe
            if (deltaX > 0 && direction.x === 0) {
                direction = { x: 1, y: 0 }; // Swipe right
            } else if (deltaX < 0 && direction.x === 0) {
                direction = { x: -1, y: 0 }; // Swipe left
            }
        } else {
            // Vertical swipe
            if (deltaY > 0 && direction.y === 0) {
                direction = { x: 0, y: 1 }; // Swipe down
            } else if (deltaY < 0 && direction.y === 0) {
                direction = { x: 0, y: -1 }; // Swipe up
            }
        }

        if (!gameStarted) {
            gameStarted = true;
            updateGameInterval(); // Start the game with the initial interval
        }
    };

    const drawGame = () => {
        // Clear the canvas
        ctx.clearRect(0, 0, canvasSize, canvasSize);

        // Draw the grid
        drawGrid();

        // Draw the snake and food
        drawSnake();
        drawFood();
    };

    onMount(() => {
        ctx = canvas.getContext('2d')!;

        resetGame(); // Initialize the game with a random snake position
        window.addEventListener('keydown', handleKeyDown);

        // Add touch event listeners for mobile support
        canvas.addEventListener('touchstart', handleTouchStart);
        canvas.addEventListener('touchend', handleTouchEnd);
    });

    onDestroy(() => {
        if (gameInterval) {
            clearInterval(gameInterval);
        }
        if (browser) {
            window.removeEventListener('keydown', handleKeyDown);
            canvas.removeEventListener('touchstart', handleTouchStart);
            canvas.removeEventListener('touchend', handleTouchEnd);
        }
    });
</script>

<div class="snake">
    <canvas bind:this={canvas} width={canvasSize} height={canvasSize}></canvas>
    <div class="controls">
        {#if gameStarted}
            <span>Score: {snake.length}</span>
        {:else}
            <span>Swipe or use arrow keys to move.</span>
        {/if}
    </div>
</div>

<style>
    .snake {
        width: 300px;
    }

    .snake canvas {
        width: 300px;
        height: 300px;
    }

    .controls {
        width: 100%;
        margin-top: 10px;
    }

    canvas {
        border: 1px solid rgba(0, 0, 0, 0.05);
        touch-action: none; /* Disable default touch behaviors like scrolling */
    }
</style>