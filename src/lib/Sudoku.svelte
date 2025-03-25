<script lang="ts">
    import { onMount } from "svelte";
    import { getSudoku } from "sudoku-gen";

    let startedAt = Date.now();
    let game = $state(getSudoku("easy"));
    let values: string = $state(game.puzzle);
    let record: {wins: number; fastest: number;} = $state({wins: 0, fastest: 0});
    let won: boolean = $state(false);

    function reset() {
        game = getSudoku("easy");
        values = game.puzzle;
        startedAt = Date.now();
        won = false;
    }

    function winCheck() {
        if (values !== game.solution) return;

        record.wins++;
        const time = Date.now() - startedAt;
        if (time < record.fastest || record.fastest === 0) record.fastest = time;

        localStorage.setItem('sudoku-record', JSON.stringify(record));
        won = true;
    }

    function handleInput(e: Event, row: number, col: number): void {
        if (!e.target) return;

        let target = e.target as HTMLInputElement;
        let value = target.value;

        if (!value || isNaN(Number(value)) || value.slice(-1) === "0") {
            target.value = "-";
            value = "-";
        }

        values = values.slice(0, (row*9) + col) + value + values.slice((row*9) + col + 1);
        target.value = value.slice(-1);
        
        if (!values.includes("-")) winCheck();
    }

    onMount(() => {
        
        // Get record from local storage
        const storedRecord = localStorage.getItem('sudoku-record');
        if (storedRecord) {
            try {
                const parsedRecord = JSON.parse(storedRecord);

                // Check schema
                if (parsedRecord.wins && parsedRecord.fastest) {
                    record = {wins: parsedRecord.wins, fastest: parsedRecord.fastest};
                } else {
                    localStorage.removeItem('sudoku-record');
                }

            } catch (error) {
                localStorage.removeItem('sudoku-record');
            }
        }

        // If no record found, initialize
        if (!record) {
            record = { wins: 0, fastest: 0 };
        }

    });
</script>

<!-- Game UI -->
<div class="sudoku">
    <div class="board">
        <table>
            <tbody>
                {#each Array(9) as _, row}
                    <tr>
                        {#each Array(9) as _, col}
                            {@const cell = game.puzzle.split("")[(row*9) + col]}
                            {@const value = values.split("")[(row*9) + col]}
                            <td>
                                <input type="number" min="1" max="9" maxlength="2" pattern="[0-9]*" value={value} disabled={cell !== "-"} oninput={(event: Event)=>handleInput(event, row, col)} />
                            </td>
                        {/each}
                    </tr>
                {/each}
            </tbody>
        </table>
    </div>

    <div class="controls">
        {#if record}
            <p>
                <span>Solves: {record.wins}</span>
            </p>
        {/if}

        {#if won}
            <span>Winner!</span>
            <button onclick={reset}>Play again</button>
        {/if}
    </div>

</div>

<!-- Styles -->
<style>
    .sudoku {
        min-width: 300px;
    }

    .sudoku .board {
        min-height: 300px;
        min-width: 300px;
        width: 100%;
        aspect-ratio: 1;
        position: relative;
        
    }

    .controls {
        width: 100%;
        margin-top: 10px;
        font-size: 14px;
        opacity: 70%;
        display: flex;
        justify-content: space-between;
    }

    table {
        border-collapse: collapse;
        width: 100%;
        height: 100%;
    }

    td {
        width: 11.11%;
        height: 11.11%;
        border: 1px solid #ddd;
        cursor: pointer;
        background-color: white;
        padding: 0;
    }

    td:hover {
        background-color: #f1f1f1;
    }

    td:nth-child(3n):not(:nth-child(9n)) {
        border-right: 1px solid #bbb;
    }

    tr:nth-child(3n):not(:nth-child(9n)) td {
        border-bottom: 1px solid #bbb;
    }

    input {
        width: 100%;
        height: 100%;
        border: none;
        background-color: transparent;
        text-align: center;
        font-size: 1.2rem;
        margin: 0;
        padding: 0;
    }

    input:disabled {
        background-color: #f1f1f1;
    }

    input:focus {
        outline: none;
    }

    input[type="number"] {
        appearance: textfield;
        -moz-appearance: textfield;
    }

    input[type="number"]::-webkit-outer-spin-button,
    input[type="number"]::-webkit-inner-spin-button {
        appearance: none;
        -webkit-appearance: none;
    }
</style>
