<script lang="ts">
    import "../app.css";
    import { HighlightSvelte } from "svelte-highlight";
    import github from "svelte-highlight/styles/github";
    import Select from 'svelte-select';

    import Snap from "$lib/Snap.svelte";
    import TicTacToe from "$lib/TicTacToe.svelte";
    import Snake from "$lib/Snake.svelte";
    import Sudoku from "$lib/Sudoku.svelte";
    import Platformer from "$lib/Platformer.svelte";
    import Two048 from "$lib/Two048.svelte";

    const games: {
        [key: string]: {        // Game name (Same as the import/file name)
            label?: string,     // Optional label for the dropdown
            emoji: string,      // emoji for the dropdown
            component: any,     // Component
            published: boolean, // Whether the game is published or not
            demoOptions?: {[key: string]: any},     // Optional options for the demo
            options?: {[key: string]: any},         // Optional options for the game
        }
    } = {
        "Snap": {
            component: Snap,
            published: true,
            emoji: "black_joker",
            demoOptions: {
                useEmoji: true,
            },
            options: {
                useEmoji: 'boolean',
                images: 'string[]',
            }
        },
        "TicTacToe": {
            component: TicTacToe,
            published: true,
            emoji: "o",
            options: {
                name: 'string',
            }
        },
        "Snake": {
            component: Snake,
            published: true,
            emoji: "snake"
        },
        "Sudoku": {
            component: Sudoku,
            published: true,
            emoji: "abacus"
        },
        "Platformer": {
            component: Platformer,
            published: false,
            emoji: "video_game"
        },
        "Two048": {
            label: "2048",
            component: Two048,
            published: true,
            emoji: "1234"
        },
    }; 

    let selector: {
        index: number,
        value: string,
        label: string,
    } = $state({
        index: 0,
        value: "Two048",
        label: "Two048",
    });
</script>

<svelte:head>

    <title>Svelte Games</title>
    <meta name="description" content="A collection of game components built with Svelte." />
    <meta name="keywords" content="svelte, games, collection" />
    <meta name="author" content="DoodadLabs" />
    
    <!-- Images -->
    <meta name="twitter:card" content="summary_large_image">
    <meta property="og:image" content="https://opengraph.githubassets.com/2148a076ddf0c2391d02189007bce78de84603d9494f4f5d70586469627814af/doodad-labs/svelte-games">
    <meta name="twitter:image" content="https://opengraph.githubassets.com/2148a076ddf0c2391d02189007bce78de84603d9494f4f5d70586469627814af/doodad-labs/svelte-games">
    <meta property="og:image:alt" content="A collection of game components built with Svelte.">

    {@html github}
</svelte:head>

<div class="w-screen h-screen flex justify-center sm:items-center bg-white sm:bg-gray-100">
    <div class="flex flex-col justify-between sm:justify-baseline gap-2 w-full sm:w-auto">
        
        <div class="sm:w-[calc(350px+2rem)] flex flex-col gap-4 bg-white pt-6 sm:pt-4 p-4 rounded-lg sm:border sm:border-gray-200">
        
            <Select class="z-50" clearable={false} showChevron bind:value={selector} items={Object.keys(games).map(key => ({ value: key, label: games[key].label || key }))}>

                <div slot="selection" class="flex place-items-center gap-2" let:selection>
                    <i class="scale-[0.9] em em-{ games[selection.value].emoji }" aria-label="{ games[selection.value].emoji }"></i>
                    {selection.label}
                </div>

                <div slot="item" class="flex place-items-center gap-2" let:item>
                    <i class="scale-[0.9] em em-{ games[item.value].emoji }" aria-label="{ games[item.value].emoji }"></i>

                    {item.label}

                    <div class="grow"></div>

                    {#if games[item.value].published}
                        <span class="text-xs {selector.value === item.value ? "bg-blue-400 text-white" : "text-blue-700 bg-blue-300/20"} px-2 py-1 rounded-md">In Production</span>
                    {:else}
                        <span class="text-xs {selector.value === item.value ? "bg-gray-200/20 text-white" : "text-gray-700 bg-gray-300/20"} px-2 py-1 rounded-md">In Development</span>
                    {/if}

                </div>
            </Select>

        </div>

        {#if games[selector.value]}
            {@const Component = games[selector.value].component}

            <div class="sm:w-[calc(350px+2rem)] overflow-auto flex flex-col gap-4 bg-white p-4 rounded-lg sm:border sm:border-gray-200">
                <Component {...games[selector.value].demoOptions}/>
            </div>
        {/if}

        <div class="hidden sm:block w-[calc(350px+2rem)] overflow-auto rounded-lg bg-white border border-gray-200">
            
            {#if games[selector.value]}
                {#if games[selector.value].published}
                    <HighlightSvelte class="w-fit overflow-auto" code={[
                        `import ${selector.value} from 'svelte-games/${selector.value}.svelte';`,
                        '',
                        `<${selector.value} ${
                            games[selector.value].options ? Object.entries(games[selector.value].options || {}).map(([key, value]) => `${key}={${value}}`).join(' ') + ' ' : ''
                        }/>`
                    ].join('\n')} />
                {:else}
                    <p class="p-4 text-gray-500">Selected game is not yet in production.</p>
                {/if}
            {:else}
                <p class="p-4 text-gray-500">Please select a game from the dropdown.</p>
            {/if}

        </div>

        <div class="flex justify-between text-sm text-gray-700 pb-4 sm:pb-0 px-4 sm:px-0">

            <span>
                Created by <a href="https://github.com/doodad-labs/" class="underline">DoodadLabs</a>
            </span>
            <span class="flex place-items-center gap-1">
                <a href="https://github.com/doodad-labs/svelte-games" class="underline">GitHub Repo</a> 
                <i class="scale-[0.8] em em-star" aria-label="star"></i>
            </span>
            
        </div>

    </div>
</div>
