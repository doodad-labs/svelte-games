<script lang="ts">
    import "../app.css";
    import { HighlightSvelte } from "svelte-highlight";
    import github from "svelte-highlight/styles/github";

    import Snap from "$lib/Snap.svelte";
    import TicTacToe from "$lib/TicTacToe.svelte";
    import Snake from "$lib/Snake.svelte";
    import Sudoku from "$lib/Sudoku.svelte";
    import Platformer from "$lib/Platformer.svelte";

    const published = [
        "Snap",
        "TicTacToe",
        "Snake",
        "Sudoku",
    ];

    let selector: string = $state("Platformer");
</script>

<svelte:head>

    <title>Svelte Games</title>
    <meta name="description" content="A collection of game components built with Svelte." />
    <meta name="keywords" content="svelte, games, collection" />
    <meta name="author" content="DoodadLabs" />
    
    <!-- Images -->
    <meta property="og:image" content="https://opengraph.githubassets.com/aa026b83ed73ca65bde41451bf0c35375c82cdea7541d98757ed56c0add4f4e0/doodad-labs/svelte-games">
    <meta name="twitter:image" content="https://opengraph.githubassets.com/aa026b83ed73ca65bde41451bf0c35375c82cdea7541d98757ed56c0add4f4e0/doodad-labs/svelte-games">
    <meta property="og:image:alt" content="A collection of game components built with Svelte.">

    {@html github}
</svelte:head>

<div class="w-screen h-screen flex justify-center sm:items-center bg-white sm:bg-gray-100">
    <div class="flex flex-col justify-between sm:justify-baseline gap-2 w-full sm:w-auto">
        
        <div class="sm:w-[calc(350px+2rem)] overflow-auto flex flex-col gap-4 bg-white pt-6 sm:pt-4 p-4 rounded-lg sm:border sm:border-gray-200">
        
            <select bind:value={selector} class="px-3 py-2 mt-0.5 w-full rounded border border-gray-200">
                <option value="Snap">Snap</option>
                <option value="TicTacToe">Tic Tac Toe</option>
                <option value="Snake">Snake</option>
                <option value="Sudoku">Sudoku</option>
                <option value="Platformer">Platformer</option>
            </select>

        </div>

        <div class="sm:w-[calc(350px+2rem)] overflow-auto flex flex-col gap-4 bg-white p-4 rounded-lg sm:border sm:border-gray-200">
            
            {#if selector === 'Snap'}
                <Snap />
            {:else if selector === 'TicTacToe'}
                <TicTacToe />
            {:else if selector === 'Snake'}
                <Snake />
            {:else if selector === 'Sudoku'}
                <Sudoku />
            {:else if selector === 'Platformer'}
                <Platformer />
            {/if}

        </div>

        <div class="hidden sm:block w-[calc(350px+2rem)] overflow-scroll rounded-lg bg-white border border-gray-200">
            
            {#if published.includes(selector)}
                <HighlightSvelte class="w-fit overflow-auto" code={[
                    /* Have to do this mess otherwise the ts compiler has a tiff */
                    '<' + 'script lang="ts"> \n',
                    `\timport ${selector} from 'svelte-games/${selector}.svelte';`, `\n`,
                    '</script> \n\n',
                    `<${selector} />`,
                ].join("")} />
            {:else}
                <div class="p-4 text-gray-500">
                    <p>Selected game is still in development and not yet in production.</p>
                </div>
            {/if}

        </div>

        <div class="flex justify-between text-sm text-gray-700 pb-4 sm:pb-0 px-4 sm:px-0">

            <span>
                Created by <a href="https://github.com/doodad-labs/" class="underline">DoodadLabs</a>
            </span>
            <span>
                <a href="https://github.com/doodad-labs/svelte-games" class="underline">GitHub Repo</a> ⭐
            </span>
            
        </div>

    </div>
</div>
