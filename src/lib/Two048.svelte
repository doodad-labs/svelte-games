<script lang="ts">
    import { BROWSER } from 'esm-env';
    import { onMount, onDestroy } from 'svelte';

    // Upscale the game for better visibility
    const canvasSize = 300 * 3;
    const gridSize = (canvasSize / 4) - 5;
    const animationDuration = 150; // milliseconds

    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;

    let loaded: boolean = $state(false);
    let gameStarted: boolean = $state(false);
    let record: number = $state(0);
    let score: number = $state(0);
    let isMobile: boolean = $state(false);
    let isFocused: boolean = $state(false);
    let gameOver: boolean = $state(false);

    const cornerRadius = 10;

    // Game state
    let grids = [
        [0, 0, 0, 0],
        [0, 0, 0, 0],
        [0, 0, 0, 0],
        [0, 0, 0, 0]
    ];

    // Animation state
    let animations: {
        value: number;
        fromX: number;
        fromY: number;
        toX: number;
        toY: number;
        progress: number;
        merged?: boolean;
        mergedFrom?: {x: number, y: number}[];
        type?: 'spawn' | 'move'; // Add this line
    }[] = $state([]);
    let lastFrameTime: number = $state(0);
    let animationRequestId: number | null = $state(null);
    let pendingNewTile: boolean = $state(false);

    // Colors for different tile values
    const tileColors: Record<number, string> = {
        0: '#cdc1b4',
        2: '#eee4da',
        4: '#ede0c8',
        8: '#f2b179',
        16: '#f59563',
        32: '#f67c5f',
        64: '#f65e3b',
        128: '#edcf72',
        256: '#edcc61',
        512: '#edc850',
        1024: '#edc53f',
        2048: '#edc22e'
    };

    // Initialise the game
    function initGame() {
        grids = [
            [0, 0, 0, 0],
            [0, 0, 0, 0],
            [0, 0, 0, 0],
            [0, 0, 0, 0]
        ];
        animations = [];
        score = 0;
        gameOver = false;
        pendingNewTile = false;
        addRandomTile();
        addRandomTile();
        render(true);
    }

    // Add a random tile (2 or 4) to an empty cell
    function addRandomTile() {
    const emptyCells: [number, number][] = [];
    
    for (let y = 0; y < 4; y++) {
        for (let x = 0; x < 4; x++) {
            if (grids[y][x] === 0) {
                emptyCells.push([x, y]);
            }
        }
    }
    
    if (emptyCells.length > 0) {
        const [x, y] = emptyCells[Math.floor(Math.random() * emptyCells.length)];
        grids[y][x] = Math.random() < 0.9 ? 2 : 4;
        
        // Add appear animation for new tile with type 'spawn'
        animations.push({
            value: grids[y][x],
            fromX: x,
            fromY: y,
            toX: x,
            toY: y,
            progress: 0,
            type: 'spawn' // Mark as spawn animation
        });
        
        // Start animation if not already running
        startAnimation();
    }
}

    // Check if the game is over
    function checkGameOver() {
        // Check if there are any empty cells
        for (let y = 0; y < 4; y++) {
            for (let x = 0; x < 4; x++) {
                if (grids[y][x] === 0) {
                    return false;
                }
            }
        }
        
        // Check if any adjacent cells can be merged
        for (let y = 0; y < 4; y++) {
            for (let x = 0; x < 4; x++) {
                if (y < 3 && grids[y][x] === grids[y + 1][x]) {
                    return false;
                }
                if (x < 3 && grids[y][x] === grids[y][x + 1]) {
                    return false;
                }
            }
        }
        
        return true;
    }

    // Animation loop
    function animate(timestamp: number) {
    if (!lastFrameTime) lastFrameTime = timestamp;
    const deltaTime = timestamp - lastFrameTime;
    lastFrameTime = timestamp;

    let allAnimationsComplete = true;

    // Update all animations
    for (let i = 0; i < animations.length; i++) {
        const anim = animations[i];
        // For spawn animations (where from and to positions are the same)
        // we want them to complete normally
        anim.progress += deltaTime / animationDuration;
        
        if (anim.progress < 1) {
            allAnimationsComplete = false;
        } else {
            anim.progress = 1;
        }
    }

    render();

    if (!allAnimationsComplete) {
        animationRequestId = requestAnimationFrame(animate);
    } else {
        animationRequestId = null;
        // Add new tile after animations complete if pending
        if (pendingNewTile) {
            pendingNewTile = false;
            addRandomTile();
            gameOver = checkGameOver();

            render();
        }
    }
}

    // Start animation if not already running
    function startAnimation() {
        if (!animationRequestId) {
            lastFrameTime = 0;
            animationRequestId = requestAnimationFrame(animate);
        }
    }

    // Render the game board
    function render(firstRender: boolean = false) {
        if (!ctx) return;
        
        // Clear canvas
        ctx.clearRect(0, 0, canvasSize, canvasSize);
        
        // Draw background
        ctx.fillStyle = '#9e948a';
        ctx.fillRect(0, 0, canvasSize, canvasSize);

        // Draw grid cells
        ctx.fillStyle = '#cdc1b4';

        for (let y = 0; y < 4; y++) {
            for (let x = 0; x < 4; x++) {
                ctx.beginPath();
                ctx.roundRect(
                    x * gridSize + 20,
                    y * gridSize + 20,
                    gridSize - 20,
                    gridSize - 20,
                    cornerRadius
                );
                ctx.fill();
            }
        }

        
        // Draw static tiles (not currently animating)
        if (firstRender) {
            for (let y = 0; y < 4; y++) {
                for (let x = 0; x < 4; x++) {
                    const value = grids[y][x];
                    // Only draw if not part of any animation
                    const isAnimating = animations.some(anim => 
                        (anim.toX === x && anim.toY === y && anim.progress === 1 && !anim.merged) ||
                        (anim.mergedFrom && anim.mergedFrom.some(pos => pos.x === x && pos.y === y))
                    );
                    if (value > 0 && !isAnimating) {
                        drawTile(x, y, value);
                    }
                }
            }
        }
        
        // Draw animating tiles
        for (const anim of animations) {
            const x = anim.fromX + (anim.toX - anim.fromX) * anim.progress;
            const y = anim.fromY + (anim.toY - anim.fromY) * anim.progress;
        
            
            drawTile(x, y, anim.value);
            ctx.restore();
            
            // Draw merged tiles fading out
            if (anim.mergedFrom) {
                for (const pos of anim.mergedFrom) {
                    const mergeX = pos.x + (anim.toX - pos.x) * anim.progress;
                    const mergeY = pos.y + (anim.toY - pos.y) * anim.progress;
                    const alpha = 1 - anim.progress;
                    
                    ctx.save();
                    ctx.globalAlpha = alpha;
                    drawTile(mergeX, mergeY, anim.value / 2);
                    ctx.restore();
                }
            }
        }
        
        // Draw game over message if needed
        if (gameOver) {
            ctx.fillStyle = 'rgba(238, 228, 218, 0.73)';
            ctx.fillRect(0, 0, canvasSize, canvasSize);
            
            ctx.fillStyle = '#776e65';
            ctx.font = `bold ${gridSize / 2}px Arial`;
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText('Game Over!', canvasSize / 2, canvasSize / 2 - 60);
            
            ctx.font = `bold ${gridSize / 4}px Arial`;
            ctx.fillText('Click to play again', canvasSize / 2, canvasSize / 2 + 60);
        }
    }

    // Helper function to draw a tile
    function drawTile(x: number, y: number, value: number) {
    // Check if this is a spawn animation
    const spawnAnim = animations.find(anim => 
        anim.type === 'spawn' && 
        anim.toX === x && 
        anim.toY === y && 
        anim.progress < 1
    );
    
    ctx.save();
    
    if (spawnAnim) {
        // Apply scaling transform only for spawn animations
        const scale = spawnAnim.progress;
        const centerX = x * gridSize + 20 + (gridSize - 20) / 2;
        const centerY = y * gridSize + 20 + (gridSize - 20) / 2;
        ctx.translate(centerX, centerY);
        ctx.scale(scale, scale);
        ctx.translate(-centerX, -centerY);
    }
    
    // Draw tile background
    ctx.fillStyle = tileColors[value] || '#3c3a32';

    ctx.beginPath();
    ctx.roundRect(
        x * gridSize + 20,
        y * gridSize + 20,
        gridSize - 20,
        gridSize - 20,
        cornerRadius
    );
    ctx.fill();
    
    // Draw tile value
    ctx.fillStyle = value <= 4 ? '#776e65' : '#f9f6f2';
    const fontSize = gridSize / 3;
    ctx.font = `bold ${fontSize}px Arial`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(
        value.toString(),
        ((x * gridSize) + 10) + gridSize / 2,
        ((y * gridSize) + 10) + gridSize / 2
    );
    
    ctx.restore();
}

    // Move tiles in a direction
    function move(direction: 'up' | 'down' | 'left' | 'right') {
        if (gameOver || animationRequestId !== null) return false;
        
        let moved = false;
        const oldGrid = JSON.parse(JSON.stringify(grids));
        const newGrid = JSON.parse(JSON.stringify(grids));
        animations = [];
        
        if (direction === 'left' || direction === 'right') {
            for (let y = 0; y < 4; y++) {
                let row = newGrid[y].filter((val: number) => val !== 0);
                const originalPositions: number[] = [];
                
                // Track original positions for animation
                for (let x = 0; x < 4; x++) {
                    if (oldGrid[y][x] !== 0) {
                        originalPositions.push(x);
                    }
                }
                
                if (direction === 'right') {
                    row.reverse();
                }
                
                // Merge tiles and track merged positions
                const mergedIndices: number[] = [];
                for (let x = 0; x < row.length - 1; x++) {
                    if (row[x] === row[x + 1] && !mergedIndices.includes(x)) {
                        row[x] *= 2;
                        row[x + 1] = 0;
                        mergedIndices.push(x);
                        moved = true;
                    }
                }
                
                // Remove zeros
                row = row.filter((val: number) => val !== 0);
                
                // Pad with zeros
                while (row.length < 4) {
                    row.push(0);
                }
                
                if (direction === 'right') {
                    row.reverse();
                }
                
                newGrid[y] = row;
                
                // Create animations
                let newPositions: number[] = [];
                for (let x = 0; x < 4; x++) {
                    if (newGrid[y][x] !== 0) {
                        newPositions.push(x);
                    }
                }
                
                let originalIndex = 0;
                for (let newIndex = 0; newIndex < newPositions.length; newIndex++) {
                    const newX = newPositions[newIndex];
                    const newValue = newGrid[y][newX];
                    
                    // Check if this is a merged tile
                    if (originalIndex < originalPositions.length - 1 && 
                        newValue === oldGrid[y][originalPositions[originalIndex]] * 2 &&
                        oldGrid[y][originalPositions[originalIndex]] === oldGrid[y][originalPositions[originalIndex + 1]]) {
                        
                        // Create merge animation
                        animations.push({
                            value: newValue,
                            fromX: originalPositions[originalIndex],
                            fromY: y,
                            toX: newX,
                            toY: y,
                            progress: 0,
                            merged: true,
                            mergedFrom: [
                                {x: originalPositions[originalIndex], y: y},
                                {x: originalPositions[originalIndex + 1], y: y}
                            ]
                        });
                        originalIndex += 2; // Skip the next tile as it's merged with this one
                        moved = true;
                    } else if (originalIndex < originalPositions.length) {
                        // Regular movement
                        if (originalPositions[originalIndex] !== newX || oldGrid[y][originalPositions[originalIndex]] !== newValue) {
                            moved = true;
                        }
                        animations.push({
    value: newValue,
    fromX: originalPositions[originalIndex],
    fromY: y,
    toX: newX,
    toY: y,
    progress: 0,
    type: 'move' // Mark as move animation
});
                        originalIndex++;
                    }
                }
            }
        } else { // up or down
            for (let x = 0; x < 4; x++) {
                let column = [
                    newGrid[0][x], 
                    newGrid[1][x], 
                    newGrid[2][x], 
                    newGrid[3][x]
                ].filter(val => val !== 0);
                
                const originalPositions: number[] = [];
                
                // Track original positions for animation
                for (let y = 0; y < 4; y++) {
                    if (oldGrid[y][x] !== 0) {
                        originalPositions.push(y);
                    }
                }
                
                if (direction === 'down') {
                    column.reverse();
                }
                
                // Merge tiles and track merged positions
                const mergedIndices: number[] = [];
                for (let y = 0; y < column.length - 1; y++) {
                    if (column[y] === column[y + 1] && !mergedIndices.includes(y)) {
                        column[y] *= 2;
                        column[y + 1] = 0;
                        mergedIndices.push(y);
                        moved = true;
                    }
                }
                
                // Remove zeros
                column = column.filter(val => val !== 0);
                
                // Pad with zeros
                while (column.length < 4) {
                    column.push(0);
                }
                
                if (direction === 'down') {
                    column.reverse();
                }
                
                // Update grid
                for (let y = 0; y < 4; y++) {
                    newGrid[y][x] = column[y];
                }
                
                // Create animations
                let newPositions: number[] = [];
                for (let y = 0; y < 4; y++) {
                    if (newGrid[y][x] !== 0) {
                        newPositions.push(y);
                    }
                }
                
                let originalIndex = 0;
                for (let newIndex = 0; newIndex < newPositions.length; newIndex++) {
                    const newY = newPositions[newIndex];
                    const newValue = newGrid[newY][x];
                    
                    // Check if this is a merged tile
                    if (originalIndex < originalPositions.length - 1 && 
                        newValue === oldGrid[originalPositions[originalIndex]][x] * 2 &&
                        oldGrid[originalPositions[originalIndex]][x] === oldGrid[originalPositions[originalIndex + 1]][x]) {
                        
                        // Create merge animation
                        animations.push({
                            value: newValue,
                            fromX: x,
                            fromY: originalPositions[originalIndex],
                            toX: x,
                            toY: newY,
                            progress: 0,
                            merged: true,
                            mergedFrom: [
                                {x: x, y: originalPositions[originalIndex]},
                                {x: x, y: originalPositions[originalIndex + 1]}
                            ]
                        });
                        originalIndex += 2; // Skip the next tile as it's merged with this one
                        moved = true;
                    } else if (originalIndex < originalPositions.length) {
                        // Regular movement
                        if (originalPositions[originalIndex] !== newY || oldGrid[originalPositions[originalIndex]][x] !== newValue) {
                            moved = true;
                        }
                        animations.push({
    value: newValue,
    fromX: x,
    fromY: originalPositions[originalIndex],
    toX: x,
    toY: newY,
    progress: 0,
    type: 'move' // Mark as move animation
});
                        originalIndex++;
                    }
                }
            }
        }

        // Update the grid only if movement occurred
        if (moved) {
            grids = newGrid;
            
            // Update score with the maximum tile value
            const maxTile = Math.max(...grids.flat());
            if (maxTile > score) {
                score = maxTile;

                if (record < score && BROWSER) {
                    record = score;
                    localStorage.setItem('2048-record', record.toString())
                }
            }
            
            pendingNewTile = true;
            startAnimation();
        }
        
        return moved;
    }

    // Handle keyboard input
    function handleKeyDown(e: KeyboardEvent) {
        if (!isFocused) return;
        if (animationRequestId !== null) return;
        
        if (!gameStarted) {
            if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(e.code)) {
                gameStarted = true;
                initGame();
            }
            return;
        }
        
        if (gameOver && (e.code === 'Space' || e.key === ' ')) {
            initGame();
            return;
        }
        
        switch (e.key) {
            case 'ArrowUp':
                move('up');
                e.preventDefault();
                break;
            case 'ArrowDown':
                move('down');
                e.preventDefault();
                break;
            case 'ArrowLeft':
                move('left');
                e.preventDefault();
                break;
            case 'ArrowRight':
                move('right');
                e.preventDefault();
                break;
        }
    }

    // Handle touch events for mobile
    let touchStartX = 0;
    let touchStartY = 0;

    function handleTouchStart(e: TouchEvent) {
        if (!gameStarted) {
            gameStarted = true;
            initGame();
            return;
        }
        
        if (gameOver) {
            initGame();
            return;
        }
        
        touchStartX = e.touches[0].clientX;
        touchStartY = e.touches[0].clientY;
    }

    function handleTouchEnd(e: TouchEvent) {
        if (!gameStarted || gameOver || animationRequestId !== null) return;
        
        const touchEndX = e.changedTouches[0].clientX;
        const touchEndY = e.changedTouches[0].clientY;
        
        const dx = touchEndX - touchStartX;
        const dy = touchEndY - touchStartY;
        
        // Only consider it a swipe if the movement is significant enough
        if (Math.abs(dx) < 50 && Math.abs(dy) < 50) return;
        
        // Determine the direction of the swipe
        if (Math.abs(dx) > Math.abs(dy)) {
            if (dx > 0) {
                move('right');
            } else {
                move('left');
            }
        } else {
            if (dy > 0) {
                move('down');
            } else {
                move('up');
            }
        }
    }

    onMount(() => {
        if (!BROWSER) return;

        // Get record from local storage
        const storedRecord = localStorage.getItem('2048-record');
        if (storedRecord) {
            record = parseInt(storedRecord);
        }

        // Check if mobile
        isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);

        // Initialise canvas
        ctx = canvas.getContext('2d')!;
        loaded = true;

        // Initialise game
        gameStarted = true;
        initGame();

        // Initial render
        render();
    });

    onDestroy(() => {
        if (BROWSER) {
            if (animationRequestId) {
                cancelAnimationFrame(animationRequestId);
            }
        }
    });
</script>

<svelte:window onkeydown={handleKeyDown} />

<div class="two048">
    <div class="container">
        <canvas
            class="{isFocused ? 'focused' : ''} {loaded ? '' : 'hidden'}"
            onmouseleave={() => isFocused = false}
            onmouseenter={() => isFocused = true}
            ontouchstart={handleTouchStart}
            ontouchend={handleTouchEnd}
            bind:this={canvas}
            width={canvasSize}
            height={canvasSize}
        ></canvas>
        <svg class="{loaded ? 'hidden' : ''}" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
    </div>
    <div class="controls">
        {#if gameStarted}
            <div>
                {#if gameOver}
                    <span class="game-over">Game Over!</span>
                {:else}
                    <span>Score: {score}</span>
                {/if}
            </div>
        {:else}
            <div>
                {#if isMobile}
                    <span>Swipe to move tiles</span>
                {:else}
                    <span>Use arrow keys while hovering</span>
                {/if}
            </div>
        {/if}

        {#if record}
            <span>Record: {record}</span>
        {/if}
    </div>
</div>

<style>
    .two048 {
        min-width: 300px;
    }

    .two048 .container {
        min-width: 300px;
        min-height: 300px;
        width: 100%;
        aspect-ratio: 1;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .two048 svg {
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

    .two048 canvas {
        min-width: 300px;
        min-height: 300px;
        width: 100%;
        aspect-ratio: 1;
        background: white;
        border-radius: 0.25rem;
        border: 1px solid rgba(0, 0, 0, 0.05);
        touch-action: none; /* Disable default touch behaviors like scrolling */
    }

    .two048 canvas.focused {
        border-color: #2196F3;
    }

    .two048 .hidden {
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