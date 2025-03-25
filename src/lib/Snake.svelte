<script lang="ts">
    import { BROWSER } from 'esm-env';
    import { onMount, onDestroy } from 'svelte';

    // @ts-ignore Import images
    import apple from './images/snake/apple.png'; // @ts-ignore
    import grapes from './images/snake/grapes.png'; // @ts-ignore
    import pineapple from './images/snake/pineapple.png'; // @ts-ignore
    import strawberry from './images/snake/strawberry.png'; // @ts-ignore

    let foods: HTMLImageElement[] = $state([]);

    // @ts-ignore Snake assets by Clear_code (https://opengameart.org/content/snake-game-assets)
    import body_bottomleft from './images/snake/body_bottomleft.png'; // @ts-ignore
    import body_bottomright from './images/snake/body_bottomright.png'; // @ts-ignore
    import body_horizontal from './images/snake/body_horizontal.png'; // @ts-ignore
    import body_topleft from './images/snake/body_topleft.png'; // @ts-ignore
    import body_topright from './images/snake/body_topright.png'; // @ts-ignore
    import body_vertical from './images/snake/body_vertical.png'; // @ts-ignore
    import head_down from './images/snake/head_down.png'; // @ts-ignore
    import head_left from './images/snake/head_left.png'; // @ts-ignore
    import head_right from './images/snake/head_right.png'; // @ts-ignore
    import head_up from './images/snake/head_up.png'; // @ts-ignore
    import tail_down from './images/snake/tail_down.png'; // @ts-ignore
    import tail_left from './images/snake/tail_left.png'; // @ts-ignore
    import tail_right from './images/snake/tail_right.png'; // @ts-ignore
    import tail_up from './images/snake/tail_up.png'; // @ts-ignore

    let snake_parts: {
        body: {
            bottomleft: HTMLImageElement;
            bottomright: HTMLImageElement;
            horizontal: HTMLImageElement;
            topleft: HTMLImageElement;
            topright: HTMLImageElement;
            vertical: HTMLImageElement;
        };
        head: {
            down: HTMLImageElement;
            left: HTMLImageElement;
            right: HTMLImageElement;
            up: HTMLImageElement;
        };
        tail: {
            down: HTMLImageElement;
            left: HTMLImageElement;
            right: HTMLImageElement;
            up: HTMLImageElement;
        };
    }

    // Upscale the game for better visibility
    const canvasSize = 300 * 2;
    const gridSize = 25 * 2;
    const gridCount = canvasSize / gridSize;

    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;
    let snake: { x: number; y: number }[] = $state([]);
    let food: {x: number; y: number; food: number;} = { x: 5, y: 5, food: 0 };
    let direction = { x: 0, y: 0 };
    let gameInterval: number | null = null;
    let gameStarted: boolean = $state(false);
    let baseSpeed = 200; // Base speed in milliseconds
    let speedMultiplier: number = $state(1); // Multiplier to adjust speed based on snake length
    let touchStartX = 0; // X coordinate of touch start
    let touchStartY = 0; // Y coordinate of touch start
    let record: number = $state(0)
    let loaded: boolean = $state(false)

    const drawRect = (x: number, y: number, color: string) => {
        ctx.fillStyle = color;
        ctx.fillRect(x * gridSize, y * gridSize, gridSize, gridSize);
    };

    const drawFood = () => {

        if (foods[food.food]) {
            // Draw the food image at the food's position
            ctx.drawImage(
                foods[food.food],
                (food.x * gridSize)+4,
                (food.y * gridSize)+4,
                gridSize-8,
                gridSize-8
            );
        } else {
            // Fallback to drawing a red rectangle if the image is not loaded
            drawRect(food.x, food.y, 'red');
        }
    };

    const drawSnake = () => {
        if (snake_parts === undefined) return;

        snake.forEach((segment, index) => {
            if (index === 0) {
                // Draw the head
                drawSnakeHead(segment);
            } else if (index === snake.length - 1) {
                // Draw the tail
                drawSnakeTail(segment);
            } else {
                // Draw the body
                drawSnakeBody(segment, index);
            }
        });
    };

    const drawSnakeHead = (segment: { x: number; y: number }) => {
        let headImage: HTMLImageElement;
        switch (true) {
            case direction.x === 1: // Right
                headImage = snake_parts.head.right;
                break;
            case direction.x === -1: // Left
                headImage = snake_parts.head.left;
                break;
            case direction.y === 1: // Down
                headImage = snake_parts.head.down;
                break;
            case direction.y === -1: // Up
                headImage = snake_parts.head.up;
                break;
            default:
                headImage = snake_parts.head.right; // Default to right
        }
        ctx.drawImage(headImage, segment.x * gridSize, segment.y * gridSize, gridSize, gridSize);
    };

    const drawSnakeTail = (segment: { x: number; y: number }) => {
        const prevSegment = snake[snake.length - 2];
        let tailImage: HTMLImageElement;
        if (prevSegment.x > segment.x) {
            tailImage = snake_parts.tail.left; // Tail pointing right
        } else if (prevSegment.x < segment.x) {
            tailImage = snake_parts.tail.right; // Tail pointing left
        } else if (prevSegment.y > segment.y) {
            tailImage = snake_parts.tail.up; // Tail pointing down
        } else if (prevSegment.y < segment.y) {
            tailImage = snake_parts.tail.down; // Tail pointing up
        } else {
            tailImage = snake_parts.tail.right; // Default to right
        }
        ctx.drawImage(tailImage, segment.x * gridSize, segment.y * gridSize, gridSize, gridSize);
    };

    const drawSnakeBody = (segment: { x: number; y: number }, index: number) => {
        const prevSegment = snake[index - 1];
        const nextSegment = snake[index + 1];
        let bodyImage: HTMLImageElement;

        if (prevSegment.x === nextSegment.x) {
            // Vertical body part
            bodyImage = snake_parts.body.vertical;
        } else if (prevSegment.y === nextSegment.y) {
            // Horizontal body part
            bodyImage = snake_parts.body.horizontal;
        } else {
            // Corner body part
            if (
                (prevSegment.x < segment.x && nextSegment.y > segment.y) ||
                (nextSegment.x < segment.x && prevSegment.y > segment.y)
            ) {
                bodyImage = snake_parts.body.bottomleft; // Bottom-left corner
            } else if (
                (prevSegment.x > segment.x && nextSegment.y > segment.y) ||
                (nextSegment.x > segment.x && prevSegment.y > segment.y)
            ) {
                bodyImage = snake_parts.body.bottomright; // Bottom-right corner
            } else if (
                (prevSegment.x < segment.x && nextSegment.y < segment.y) ||
                (nextSegment.x < segment.x && prevSegment.y < segment.y)
            ) {
                bodyImage = snake_parts.body.topleft; // Top-left corner
            } else {
                bodyImage = snake_parts.body.topright; // Top-right corner
            }
        }

        ctx.drawImage(bodyImage, segment.x * gridSize, segment.y * gridSize, gridSize, gridSize);
    };

    const drawGrid = () => {
        // Set the background color for the grid
        ctx.fillStyle = '#add644'; // Light green background
        ctx.fillRect(0, 0, canvasSize, canvasSize);

        // Set the color for the checkerboard pattern
        ctx.fillStyle = '#a6d13c'; // Light green with transparency

        // Draw the checkerboard pattern
        for (let x = 0; x < gridCount; x++) {
            for (let y = 0; y < gridCount; y++) {
                if ((x + y) % 2 === 0) { // Alternate cells
                    ctx.fillRect(
                        x * gridSize,
                        y * gridSize,
                        gridSize,
                        gridSize
                    );
                }
            }
        }

        // Draw grid lines (optional, for better visibility)
        ctx.strokeStyle = 'rgba(0, 0, 0, 0.075)'; // Grid line color
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
            gameOver();
            return;
        }

        // Check for collision with itself
        if (snake.some(segment => segment.x === head.x && segment.y === head.y)) {
            gameOver();
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
        let newFoodPosition: { x: number; y: number, food: number };
        do {
            newFoodPosition = {
                x: Math.floor(Math.random() * gridCount),
                y: Math.floor(Math.random() * gridCount),
                food: Math.floor(Math.random() * foods.length)
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

    const gameOver = () => {
        if (snake.length > record) {
            record = snake.length;
            localStorage.setItem('snake-record', record.toString());
        }

        resetGame();
    };

    const resetGame = () => {
        // Set the snake to a random position
        const randomPosition = getRandomPosition();
        snake = [{ x: randomPosition.x, y: randomPosition.y }];
        direction = { x: 0, y: 0 };

        // additional snake parts
        snake.push({ x: randomPosition.x - 1, y: randomPosition.y });

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

    function loadImage(src: string): Promise<HTMLImageElement> {
        return new Promise((resolve, reject) => {
            const img = new Image();
            img.onload = () => resolve(img);
            img.onerror = reject;
            img.src = src;
        });
    }

    onMount(async () => {
        ctx = canvas.getContext('2d')!;

        resetGame(); // Initialize the game with a random snake position
        window.addEventListener('keydown', handleKeyDown);

        // Add touch event listeners for mobile support
        canvas.addEventListener('touchstart', handleTouchStart);
        canvas.addEventListener('touchend', handleTouchEnd);

        // get record from local storage
        const storedRecord = localStorage.getItem('snake-record');
        if (storedRecord) {
            record = parseFloat(storedRecord);
        }

        // Load food images
        await Promise.all([
            loadImage(apple),
            loadImage(grapes),
            loadImage(pineapple),
            loadImage(strawberry),
        ]).then(([a, b, c, d]) => {
            foods.push(a);
            foods.push(b);
            foods.push(c);
            foods.push(d);
        });

        // Load snake images
        await Promise.all([
            loadImage(body_bottomleft),
            loadImage(body_bottomright),
            loadImage(body_horizontal),
            loadImage(body_topleft),
            loadImage(body_topright),
            loadImage(body_vertical),
            loadImage(head_down),
            loadImage(head_left),
            loadImage(head_right),
            loadImage(head_up),
            loadImage(tail_down),
            loadImage(tail_left),
            loadImage(tail_right),
            loadImage(tail_up),
        ]).then(([a, b, c, d, e, f, g, h, i, j, k, l, m, n]) => {
            snake_parts = {
                body: {
                    bottomleft: a,
                    bottomright: b,
                    horizontal: c,
                    topleft: d,
                    topright: e,
                    vertical: f,
                },
                head: {
                    down: g,
                    left: h,
                    right: i,
                    up: j,
                },
                tail: {
                    down: k,
                    left: l,
                    right: m,
                    up: n,
                }
            };
        });

        drawGame();
        loaded = true;

    });

    onDestroy(() => {
        if (gameInterval) {
            clearInterval(gameInterval);
        }
        if (BROWSER) {
            window.removeEventListener('keydown', handleKeyDown);
            canvas.removeEventListener('touchstart', handleTouchStart);
            canvas.removeEventListener('touchend', handleTouchEnd);
        }
    });
</script>

<div class="snake">
    <div class="container">
        <canvas class="{loaded ? '' : 'hidden'}" bind:this={canvas} width={canvasSize} height={canvasSize}></canvas>
        <svg class="{loaded ? 'hidden' : ''}" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
    </div>
    <div class="controls">
        {#if gameStarted}
            <div>
                <span>Score: {snake.length}</span>
                <span>Speed: {baseSpeed * speedMultiplier}</span>
            </div>
        {:else}
            <span>Swipe or use arrow keys to move.</span>
        {/if}

        {#if record}
            <span>Record: {record}</span>
        {/if}
    </div>
</div>

<style>
    .snake {
        width: 300px;
    }

    .snake .container {
        width: 300px;
        height: 300px;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .snake svg {
        width: 1.25rem;
        height: 1.25rem;
        color: black;
        animation: spin 1s linear infinite;
    }

    @keyframes spin {
        to {
            transform: rotate(360deg);
        }
    }

    .snake canvas {
        width: 300px;
        height: 300px;
        background: white;
        border-radius: 0.25rem;
    }

    .snake .hidden {
        display: none;
    }

    .controls {
        width: 100%;
        margin-top: 10px;
        font-size: 14px;
        opacity: 70%;
        display: flex;
        justify-content: space-between;
    }

    canvas {
        border: 1px solid rgba(0, 0, 0, 0.05);
        touch-action: none; /* Disable default touch behaviors like scrolling */
    }
</style>