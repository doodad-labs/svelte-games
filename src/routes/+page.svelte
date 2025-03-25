<script lang="ts">

    import '../app.css';
    import Snap from '$lib/Snap.svelte';
    import TicTacToe from '$lib/TicTacToe.svelte';
    import Snake from '$lib/Snake.svelte';

    let selector: string = $state('snake');

</script>

<div class="w-screen h-screen flex justify-center items-center bg-gray-50">
    <div class="flex flex-col gap-4 w-75">
        <select bind:value={selector} class="px-3 py-2 mt-0.5 w-full rounded border border-gray-200 shadow-sm sm:text-sm">
            <option value="snap">Snap</option>
            <option value="tictactoe">Tic Tac Toe</option>
            <option value="snake">Snake</option>
        </select>
        
        <div>
            {#if selector === 'snap'}
                <Snap />
            {:else if selector === 'tictactoe'}
                <TicTacToe />
            {:else if selector === 'snake'}
                <Snake />
            {/if}
        </div>
        
    </div>
</div>

<div class="absolute bottom-0 left-0 p-4 flex w-screen justify-between flex-col sm:flex-row">
    <span class="text-sm">
        Created by <a href="https://github.com/doodad-labs/" class="underline text-blue-600">🧪 DoodadLabs</a>
    </span>
    <span>
        Give use a star on
        <a href="https://github.com/doodad-labs/svelte-games" class="underline text-blue-600">GitHub</a>
    </span>
</div>