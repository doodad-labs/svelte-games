<script lang="ts">
    import { BROWSER } from 'esm-env';
    import { onMount, onDestroy } from 'svelte';

    // Upscale the game for better visibility
    const canvasSize = 300 * 3;
    const gridSize = 25 * 3;

    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;

    let loaded: boolean = $state(false);
    let gameStarted: boolean = $state(false);
    let record: number = $state(0);
    let score: number = $state(0);
    let isMobile: boolean = $state(false);
    let isFocused: boolean = $state(false);

    // Game state
    let player = {
        x: gridSize * 2,
        y: canvasSize / 2,
        width: gridSize * 0.8,
        height: gridSize * 1.5,
        velocityY: 0,
        jumping: false,
        speed: 5
    };

    let platforms = $state<Array<{x: number, y: number, width: number}>>([]);
    let gameOver = $state(false);
    let animationFrameId: number;
    let scrollOffset = 0;

    // Physics constants
    const GRAVITY = 0.4;
    const JUMP_FORCE = -10;
    const PLATFORM_WIDTH = gridSize * 3;
    const PLATFORM_HEIGHT = gridSize / 2;

    function generateInitialPlatforms() {
        platforms = [];
        // Starting platform
        platforms.push({
            x: player.x - gridSize,
            y: player.y + player.height,
            width: PLATFORM_WIDTH
        });

        // Generate some initial platforms to the right
        for (let i = 0; i < 5; i++) {
            generateNextPlatform();
        }
    }

    function generateNextPlatform() {
        const lastPlatform = platforms[platforms.length - 1];
        const minDistance = gridSize * 4;
        const maxDistance = gridSize * 6;
        const distance = minDistance + Math.random() * (maxDistance - minDistance);
        
        // Random y change (-1, 0, or +1 grid sizes)
        const yChange = (Math.floor(Math.random() * 3) - 1) * gridSize;
        const newY = Math.min(
            Math.max(lastPlatform.y + yChange, gridSize * 2),
            canvasSize - gridSize * 3
        );

        platforms.push({
            x: lastPlatform.x + distance,
            y: newY,
            width: PLATFORM_WIDTH
        });
    }

    function startGame() {
        if (gameStarted) return;
        
        gameStarted = true;
        gameOver = false;
        score = 0;
        scrollOffset = 0;
        player = {
            x: gridSize * 2,
            y: canvasSize / 2,
            width: gridSize * 0.8,
            height: gridSize * 1.5,
            velocityY: 0,
            jumping: false,
            speed: 5
        };
        
        generateInitialPlatforms();
        gameLoop();
    }

    function gameLoop() {
        if (gameOver) return;

        update();
        render();
        animationFrameId = requestAnimationFrame(gameLoop);
    }

    function update() {
        // Apply gravity
        player.velocityY += GRAVITY;
        player.y += player.velocityY;

        // Move player to the right
        player.x += player.speed;
        scrollOffset += player.speed;
        score = Math.floor(scrollOffset / gridSize);

        // Check for platform collisions
        let onGround = false;
        for (const platform of platforms) {
            if (
                player.x + player.width > platform.x &&
                player.x < platform.x + platform.width &&
                player.y + player.height >= platform.y &&
                player.y + player.height <= platform.y + PLATFORM_HEIGHT &&
                player.velocityY > 0
            ) {
                player.y = platform.y - player.height;
                player.velocityY = 0;
                player.jumping = false;
                onGround = true;
            }
        }

        // Generate new platforms as needed
        while (platforms.length < 10 && platforms[platforms.length - 1].x < scrollOffset + canvasSize) {
            generateNextPlatform();
        }

        // Remove platforms that are far behind
        if (platforms.length > 0 && platforms[0].x + platforms[0].width < scrollOffset - gridSize) {
            platforms.shift();
        }

        // Check for game over (falling off screen)
        if (player.y > canvasSize) {
            endGame();
        }
    }

    function render() {
        // Clear canvas
        ctx.clearRect(0, 0, canvasSize, canvasSize);

        // Draw platforms (relative to scroll offset)
        ctx.fillStyle = '#4CAF50';
        for (const platform of platforms) {
            const platformX = platform.x - scrollOffset;
            if (platformX > -platform.width && platformX < canvasSize) {
                ctx.fillRect(
                    platformX,
                    platform.y,
                    platform.width,
                    PLATFORM_HEIGHT
                );
            }
        }

        // Draw player (fixed x position, moves through scrolling)
        const playerX = player.x - scrollOffset;
        ctx.fillStyle = '#2196F3';
        ctx.fillRect(playerX, player.y, player.width, player.height);
    }

    function jump() {
        if (!player.jumping) {
            player.velocityY = JUMP_FORCE;
            player.jumping = true;
        }
    }

    function endGame() {
        gameOver = true;
        cancelAnimationFrame(animationFrameId);
        
        // Update record if needed
        if (score > record) {
            record = score;
            localStorage.setItem('platformer-record', record.toString());
        }
    }

    onMount(() => {
        if (!BROWSER) return;

        // Get record from local storage
        const storedRecord = localStorage.getItem('platformer-record');
        if (storedRecord) {
            record = parseInt(storedRecord);
        }

        // Initialize canvas
        ctx = canvas.getContext('2d')!;
        canvas.width = canvasSize;
        canvas.height = canvasSize;
        loaded = true;

        // Check if mobile
        isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent)

        // Initial platforms
        generateInitialPlatforms();
        render();
    });

    onDestroy(() => {
        if (animationFrameId) {
            cancelAnimationFrame(animationFrameId);
        }
        if (canvas) {
            canvas.removeEventListener('touchstart', handleClick);
        }
    });

    // Handle touch events
    function handleClick() {

        if (!gameStarted) {
            return startGame()
        };

        jump();
    }

    // Handle keyboard input
    function handleKeyDown(e: KeyboardEvent) {

        if (!isFocused) return;

        if (e.code === 'Space' || e.key === 'ArrowUp') {

            if (!gameStarted) {
                return startGame()
            };

            e.preventDefault();
            jump();
        }
    }
</script>

<svelte:window onkeydown={handleKeyDown} />

<div class="platformer">
    <div class="container">
        <canvas class="{isFocused ? 'focused' : ''} {loaded ? '' : 'hidden'}" onmouseleave={()=>isFocused=false} onmouseenter={()=>isFocused=true} onclick={handleClick} bind:this={canvas} width={canvasSize} height={canvasSize}></canvas>
        <svg class="{loaded ? 'hidden' : ''}" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
    </div>
    <div class="controls">
        {#if gameStarted}
            <div>
                <span>Score: {score}</span>
            </div>
        {:else}
            {#if isMobile}
                <span>Tap to jump</span>
            {:else}
                <span>Press "space" while hovering</span>
            {/if}
        {/if}

        {#if record}
            <span>Record: {record}</span>
        {/if}
    </div>
</div>

<style>
    .platformer {
        min-width: 300px;
    }

    .platformer .container {
        min-width: 300px;
        min-height: 300px;
        width: 100%;
        aspect-ratio: 1;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .platformer svg {
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

    .platformer canvas {
        min-width: 300px;
        min-height: 300px;
        width: 100%;
        aspect-ratio: 1;
        background: white;
        border-radius: 0.25rem;
        border: 1px solid rgba(0, 0, 0, 0.05);
        touch-action: none; /* Disable default touch behaviors like scrolling */
    }

    .platformer canvas.focused {
        border-color: #2196F3;
    }

    .platformer .hidden {
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