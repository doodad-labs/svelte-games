<script lang="ts">

    import '../app.css';
    import Snap from '$lib/Snap.svelte';
    import TicTacToe from '$lib/TicTacToe.svelte';
    import Snake from '$lib/Snake.svelte';

    let selector: string = $state('snake');

</script>

<div class="w-screen h-screen flex justify-center sm:items-center bg-white sm:bg-gray-100">
    <div class="flex flex-col justify-between sm:justify-baseline gap-2">
        <div class="flex flex-col gap-4 w-75 sm:w-[calc(300px+2rem)] bg-white pt-6 sm:p-4 rounded-lg sm:border sm:border-gray-200">
        
            <select bind:value={selector} class="px-3 py-2 mt-0.5 w-75 rounded border border-gray-200">
                <option value="snap">Snap</option>
                <option value="tictactoe">Tic Tac Toe</option>
                <option value="snake">Snake</option>
            </select>
            
            <div class="w-75">
                {#if selector === 'snap'}
                    <Snap />
                {:else if selector === 'tictactoe'}
                    <TicTacToe />
                {:else if selector === 'snake'}
                    <Snake />
                {/if}
            </div>
            
        </div>
        <div class="flex justify-between text-sm text-gray-700 pb-4 sm:pb-0">
            <span>
                Created by 🧪 <a href="https://github.com/doodad-labs/" class="underline">DoodadLabs</a>
            </span>

            <span>
                <a href="https://github.com/doodad-labs/svelte-games" class="underline">GitHub Repo</a> ⭐
            </span>
        </div>
    </div>
</div>
