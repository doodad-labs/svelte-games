<script lang="ts">
    // Props
    let { 
        name, 
    }: { 
        name?: string | null
    } = $props();

    // Game state
    let cells = Array.from({ length: 9 }, (_, i) => i); // Represents the 3x3 grid
    let x_cells: number[] = $state([]); // Cells claimed by 'x'
    let o_cells: number[] = $state([]); // Cells claimed by 'o'
    let turn: 'x' | 'o' = $state('x'); // Current player's turn
    let winner: 'x' | 'o' | 'draw' | null = $state(null); // Winner of the game

    // Claim a cell for a player
    async function claim(cell: number, user: 'x' | 'o') {
        // Validate the move
        if (turn !== user) return; // Not the player's turn
        if (x_cells.includes(cell) || o_cells.includes(cell)) return; // Cell already claimed
        if (winner) return; // Game already won

        // Update the game state
        if (user === 'x') {
            x_cells = [...x_cells, cell];
            turn = 'o'; // Switch to 'o's turn
            bots_turn(); // Let the bot take its turn
        } else {
            o_cells = [...o_cells, cell];
            turn = 'x'; // Switch to 'x's turn
        }

        // Check for a winner
        winner = checkWinner();
    }

    // Check if there's a winner
    function checkWinner(): 'x' | 'o' | 'draw' | null {
        const winningCombinations = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8], // Rows
            [0, 3, 6], [1, 4, 7], [2, 5, 8], // Columns
            [0, 4, 8], [2, 4, 6] // Diagonals
        ];

        // Check if 'x' or 'o' has a winning combination
        for (const combination of winningCombinations) {
            if (combination.every(cell => x_cells.includes(cell))) return 'x';
            if (combination.every(cell => o_cells.includes(cell))) return 'o';
        }

        // Check for a draw
        if (x_cells.length + o_cells.length === 9) return 'draw';

        return null; // No winner yet
    }

    // Bot's turn logic
    async function bots_turn() {
        if (turn !== 'o') return; // Not the bot's turn
        await new Promise(resolve => setTimeout(resolve, Math.max(100, Math.min(500, Math.random()*1000)))); // Simulate thinking time (100ms - 500ms)
        if (winner) return; // Game already won

        const availableCells = cells.filter(cell => !x_cells.includes(cell) && !o_cells.includes(cell));
        if (availableCells.length === 0) return; // No available cells

        // Step 1: Check if the bot can win
        for (const cell of availableCells) {
            const testCells = [...o_cells, cell];
            if (checkWinningCombination(testCells)) {
                claim(cell, 'o');
                return;
            }
        }

        // Step 2: Block the player if they're about to win
        for (const cell of availableCells) {
            const testCells = [...x_cells, cell];
            if (checkWinningCombination(testCells)) {
                claim(cell, 'o');
                return;
            }
        }

        // Step 3: Take the center if available
        const centerCell = 4;
        if (availableCells.includes(centerCell)) {
            claim(centerCell, 'o');
            return;
        }

        // Step 4: If player has two opposite corners, force a side move to prevent fork
        const cornerCells = [0, 2, 6, 8];
        const oppositePairs = [[0, 8], [2, 6]];
        
        let hasOppositeCorners = false;
        for (const [a, b] of oppositePairs) {
            if (x_cells.includes(a) && x_cells.includes(b)) {
                hasOppositeCorners = true;
                break;
            }
        }
        
        // If player has opposite corners, take a side to prevent fork
        if (hasOppositeCorners) {
            const sideCells = [1, 3, 5, 7];
            const availableSides = sideCells.filter(cell => availableCells.includes(cell));
            if (availableSides.length > 0) {
                claim(availableSides[0], 'o'); // Take first available side (could randomize if preferred)
                return;
            }
        }

        // Step 5: Take a corner that doesn't give player a fork opportunity
        const availableCorners = cornerCells.filter(cell => availableCells.includes(cell));
        if (availableCorners.length > 0) {
            // Prefer corners adjacent to player's corners to create our own potential forks
            let bestCorner = null;
            
            // Look for corners adjacent to our existing corners
            for (const corner of availableCorners) {
                // If we have a corner, try to take an adjacent corner to create a potential fork
                if (o_cells.some(oc => Math.abs(oc - corner) === 2 || Math.abs(oc - corner) === 6)) {
                    bestCorner = corner;
                    break;
                }
            }
            
            // If no strategic corner found, take one that's not opposite to player's corner
            if (!bestCorner) {
                for (const corner of availableCorners) {
                    const oppositeCorner = 8 - corner; // 0<>8, 2<>6
                    if (!x_cells.includes(oppositeCorner)) {
                        bestCorner = corner;
                        break;
                    }
                }
            }
            
            // If all else fails, take any available corner
            if (!bestCorner) {
                bestCorner = availableCorners[0];
            }
            
            claim(bestCorner, 'o');
            return;
        }

        // Step 6: Take any available side
        const sideCells = [1, 3, 5, 7];
        const availableSides = sideCells.filter(cell => availableCells.includes(cell));
        if (availableSides.length > 0) {
            const randomSide = availableSides[Math.floor(Math.random() * availableSides.length)];
            claim(randomSide, 'o');
            return;
        }
    }

    // Helper function to check if a set of cells includes a winning combination
    function checkWinningCombination(cells: number[]): boolean {
        const winningCombinations = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8], // Rows
            [0, 3, 6], [1, 4, 7], [2, 5, 8], // Columns
            [0, 4, 8], [2, 4, 6] // Diagonals
        ];

        return winningCombinations.some(combination =>
            combination.every(cell => cells.includes(cell))
        );
    }

    // Reset the game
    function reset() {
        x_cells = [];
        o_cells = [];
        winner = null;
        turn = 'x';
    }
</script>

<!-- Game UI -->
<div class="tic-tac-toe">
    <div class="cells">
        {#each cells as cell}
            <button
                class="cell {turn === 'x' && !x_cells.includes(cell) ? 'turn' : ''}"
                onclick={() => claim(cell, 'x')}
            >
                {#if x_cells.includes(cell)}
                    <!-- X icon -->
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 256 256"><path opacity="1" d="M205.66,194.34a8,8,0,0,1-11.32,11.32L128,139.31,61.66,205.66a8,8,0,0,1-11.32-11.32L116.69,128,50.34,61.66A8,8,0,0,1,61.66,50.34L128,116.69l66.34-66.35a8,8,0,0,1,11.32,11.32L139.31,128Z"></path></svg>
                {:else if o_cells.includes(cell)}
                    <!-- O icon -->
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 256 256"><path opacity="1" d="M128,24A104,104,0,1,0,232,128,104.11,104.11,0,0,0,128,24Zm0,192a88,88,0,1,1,88-88A88.1,88.1,0,0,1,128,216Z"></path></svg>
                {/if}
            </button>
        {/each}
    </div>

    <!-- Game controls -->
    <div class="controls">

        {#if x_cells.length}
            {#if winner}
                {#if winner === 'draw'}
                    <span>It's a draw!</span>
                {:else}
                    <span class="ellipsis">
                        <span>{winner === 'x' ? (name ? name : 'You') : 'Bot'}</span> win{name ? 's' : ''}!
                    </span>
                {/if}
            {:else}
                <span class="ellipsis">
                    <span>{turn === 'x' ? (name ? `${name}'s` : 'Your') : 'Bots'}</span> turn.
                </span>
            {/if}

            <button onclick={reset}>Reset</button>
        {:else}
            <span>Click a square to start.</span>
        {/if}
    </div>
</div>

<!-- Styles -->
<style>
    .tic-tac-toe {
        min-width: 300px;
    }

    .cells {
        min-height: 300px;
        min-width: 300px;
        width: 100%;
        aspect-ratio: 1;
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        grid-template-rows: repeat(3, 1fr);
        gap: 5px;
    }

    .controls {
        display: flex;
        justify-content: space-between;
        align-items: center;
        width: 100%;
        margin-top: 10px;
        font-size: 14px;
        opacity: 70%;
    }

    .controls button {
        cursor: pointer;
        text-decoration: underline;
    }

    .controls .ellipsis {
        display: flex;
        gap: 5px;
    }

    .controls .ellipsis span {
        text-overflow: ellipsis;
        overflow: hidden;
        white-space: nowrap;
        max-width: 160px;
    }

    .cells .cell {
        cursor: default;
        display: flex;
        justify-content: center;
        align-items: center;
        border-radius: 7px;
        background: #fff;
        border: 1px solid rgba(0, 0, 0, 0.096);
        height: 100%;
        width: 100%;
    }

    .cells .cell.turn {
        cursor: pointer;
    }

    .cells .cell svg {
        width: 40%;
        height: 40%;
    }
</style>