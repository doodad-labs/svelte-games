<script lang="ts">

    import '../app.css';
    import Snap from '$lib/Snap.svelte';
    import TicTacToe from '$lib/TicTacToe.svelte';

    let selector: string = $state('snap');

</script>

<div class="w-screen h-screen flex justify-center items-center">
    <div class="flex flex-col gap-4 w-75">
        <select bind:value={selector} class="px-3 py-2 mt-0.5 w-full rounded border border-gray-200 shadow-sm sm:text-sm">
            <option value="snap">Snap</option>
            <option value="tictactoe">Tic Tac Toe</option>
        </select>
        
        <div>
            {#if selector === 'snap'}
                <Snap />
            {:else if selector === 'tictactoe'}
                <TicTacToe />
            {/if}
        </div>
        
    </div>
</div>