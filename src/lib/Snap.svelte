<script lang="ts">

    import { BROWSER } from 'esm-env';
    import { onMount } from 'svelte';

    const {
        useEmoji,
        images,
    }: {
        useEmoji?: boolean;
        images?: string[];
    } = $props();

    // const useEmoji = true;

    // @ts-ignore Import images
    import image1 from './images/snap/gem-1.png'; // @ts-ignore
    import image2 from './images/snap/gem-2.png'; // @ts-ignore
    import image3 from './images/snap/gem-3.png'; // @ts-ignore
    import image4 from './images/snap/gem-4.png'; // @ts-ignore
    import image5 from './images/snap/gem-5.png'; // @ts-ignore
    import image6 from './images/snap/gem-6.png'; // @ts-ignore
    import image7 from './images/snap/gem-7.png'; // @ts-ignore
    import image8 from './images/snap/gem-8.png';

    // @ts-ignore Import emojis
    import crying from './images/emojis/crying.png'; // @ts-ignore
    import eyes from './images/emojis/eyes.png'; // @ts-ignore
    import fire from './images/emojis/fire.png'; // @ts-ignore
    import heart from './images/emojis/heart.png'; // @ts-ignore
    import joy from './images/emojis/joy.png'; // @ts-ignore
    import poop from './images/emojis/poop.png'; // @ts-ignore
    import popper from './images/emojis/popper.png'; // @ts-ignore
    import rocket from './images/emojis/rocket.png'; // @ts-ignore
    import skull from './images/emojis/skull.png'; // @ts-ignore
    import sparkles from './images/emojis/sparkles.png'; // @ts-ignore
    import star from './images/emojis/star.png'; // @ts-ignore

    const useCustomImages = images && images.length >= 8;

    // Array of images
    let Emojis = [crying, eyes, fire, heart, joy, poop, popper, rocket, skull, sparkles, star];
    let Images = [image1, image2, image3, image4, image5, image6, image7, image8];
    let Cards = useCustomImages ? getRandomElements(images, 8) : ( useEmoji ? getRandomElements(Emojis, 8) : Images );

    // State variables
    let flipped: number[] = $state([]); // Tracks currently flipped cards
    let matched: number[] = $state([]); // Tracks matched cards
    let cards: number[] = $state(shuffle(Array.from({ length: 16 }, (_, i) => i))); // Card indices
    let startTime: number | null = $state(null); // Start time
    let timer: number | null = $state(null); // Timer
    let timerInterval: null | number; // Timer interval
    let record: number = $state(0); // Record time

    function getRandomElements(arr: string[], count: number) {
        // Make a copy of the original array to avoid modifying it
        const shuffled = [...arr];
        
        // Fisher-Yates shuffle algorithm
        for (let i = shuffled.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
        }
        
        // Return the first 'count' elements
        return shuffled.slice(0, count);
    }

    /* 
        completely over engineered shuffle function that ensures no two adjacent cards are touching
        (horizontally, vertically, or diagonally), we could just .sort(() => Math.random()); but
        thats boring.
    */
    function shuffle(cards: number[]): number[] {
        const cardsAreTouching = (card1: number, card2: number) => {
            // Cards are considered touching if their values modulo 8 are equal
            return card1 % 8 === card2 % 8;
        };

        // Fisher-Yates shuffle
        for (let i = cards.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1)); // Random index from 0 to i
            [cards[i], cards[j]] = [cards[j], cards[i]]; // Swap elements
        }

        // Ensure no two adjacent cards are touching (horizontally, vertically, or diagonally)
        let isTouching = true;
        while (isTouching) {
            isTouching = false;
            for (let i = 0; i < cards.length; i++) {
                const row = Math.floor(i / 4); // Current row in the 4x4 grid
                const col = i % 4; // Current column in the 4x4 grid

                // Check card above
                if (row > 0 && cardsAreTouching(cards[i], cards[i - 4])) {
                    isTouching = true;
                    break;
                }
                // Check card below
                if (row < 3 && cardsAreTouching(cards[i], cards[i + 4])) {
                    isTouching = true;
                    break;
                }
                // Check card to the left
                if (col > 0 && cardsAreTouching(cards[i], cards[i - 1])) {
                    isTouching = true;
                    break;
                }
                // Check card to the right
                if (col < 3 && cardsAreTouching(cards[i], cards[i + 1])) {
                    isTouching = true;
                    break;
                }
                // Check card diagonally top-left
                if (row > 0 && col > 0 && cardsAreTouching(cards[i], cards[i - 5])) {
                    isTouching = true;
                    break;
                }
                // Check card diagonally top-right
                if (row > 0 && col < 3 && cardsAreTouching(cards[i], cards[i - 3])) {
                    isTouching = true;
                    break;
                }
                // Check card diagonally bottom-left
                if (row < 3 && col > 0 && cardsAreTouching(cards[i], cards[i + 3])) {
                    isTouching = true;
                    break;
                }
                // Check card diagonally bottom-right
                if (row < 3 && col < 3 && cardsAreTouching(cards[i], cards[i + 5])) {
                    isTouching = true;
                    break;
                }
            }

            // If any cards are touching, reshuffle
            if (isTouching) {
                shuffle(cards);
            }
        }

        return cards;
    }

    // Reset the game
    function reset() {
        flipped = [];
        matched = [];
        cards = shuffle(cards);
    }

    // Handle card flip
    function flip(card: number) {

        if (startTime === null) {
            startTimer();
        }

        if (matched.includes(card)) return; // Ignore if card is already matched

        // Reset flipped cards if two are already flipped
        if (flipped.length === 2) {
            flipped = [];
        }

        flipped = [...flipped, card]; // Add the card to flipped list
        const currentFlipped = flipped; // Store the current flipped state

        // Check for a match when two cards are flipped
        if (flipped.length === 2) {
            if (flipped[0] % 8 === flipped[1] % 8) { // If cards match
                console.log('matched');
                matched = [...matched, ...flipped]; // Add to matched list

                // Reset the game if all cards are matched
                if (matched.length === cards.length) {
                    
                    // Update record time
                    if (timer && (!record || timer < record)) {
                        record = timer;

                        // Save record to local storage
                        if (BROWSER) {
                            localStorage.setItem('snap-record', record.toString());
                        }

                    }

                    stopTimer();
                    setTimeout(() => reset(), 750);
                }
                return;
            }

            // Flip back if cards don't match
            setTimeout(() => {
                if (flipped !== currentFlipped) return; // Ensure state hasn't changed
                flipped = [];
            }, 500);
        }
    }

    function startTimer() {
        startTime = Date.now();

        timerInterval = setInterval(() => {
            if (!startTime) return;
            timer = (Date.now() - startTime) / 1000;
        }, 1000 / 60 / 2);
    }

    function stopTimer() {
        if (timerInterval) clearInterval(timerInterval);
        startTime = null;
        timerInterval = null;
    }

    onMount(() => {
        
        if (!BROWSER) return;
        
        // Get record from local storage
        const storedRecord = localStorage.getItem('snap-record');
        if (storedRecord) {
            record = parseFloat(storedRecord);
        }

    });

</script>

<!-- Preload Cards -->
<svelte:head>
    {#each Cards as image}
        <link rel="preload" href={image} as="image" />
    {/each}
</svelte:head>

<!-- Card Grid -->
<div class="snap">
    <div class="cards">
    
        {#each cards as card}
            <button
                data-update={timer /* There is a weird bug in firefox (typical) where the front question mark wont show after flip, this updates the element constantly fixing that */} 
                class="card {flipped.includes(card) || matched.includes(card) ? 'flip' : ''}"
                onclick="{() => flip(card)}"
            >
                <!-- Front of the card -->
                <div class="view front-view">
                    <svg xmlns="http://www.w3.org/2000/svg" width="43" height="43" fill="currentColor" viewBox="0 0 256 256"><path opacity="1" d="M192,96c0,28.51-24.47,52.11-56,55.56V160a8,8,0,0,1-16,0V144a8,8,0,0,1,8-8c26.47,0,48-17.94,48-40s-21.53-40-48-40S80,73.94,80,96a8,8,0,0,1-16,0c0-30.88,28.71-56,64-56S192,65.12,192,96Zm-64,96a16,16,0,1,0,16,16A16,16,0,0,0,128,192Z"></path></svg>
                </div>
    
                <!-- Back of the card -->
                <div class="view back-view">
                    {#if flipped.includes(card) || matched.includes(card)}
                        <img src={Cards[card % 8]} alt="card-img" />
                    {:else}
                        <span>Nice try! If you know what you're doing, why not contribute? <a href="https://github.com/doodad-labs/svelte-games">GitHub</a></span>
                    {/if}
                </div>
            </button>
        {/each}
    </div>
    
    <div class="controls">
        {#if timer}
            <span>Time: {timer.toFixed(2)}s</span>
        {:else}
            <span>Click on a card to flip it.</span>
        {/if}

        {#if record}
            <span>Record: {record.toFixed(2)}s</span>
        {/if}
    </div>
</div>

<!-- Styles -->
<style>

    .snap {
        min-width: 300px;
    }

    .controls {
        width: 100%;
        margin-top: 10px;
        font-size: 14px;
        opacity: 70%;
        display: flex;
        justify-content: space-between;
    }

    .cards {
        min-height: 300px;
        min-width: 300px;
        width: 100%;
        aspect-ratio: 1;
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        grid-template-rows: repeat(4, 1fr);
        gap: 5px;
    }

    .cards .card {
        cursor: pointer;
        list-style: none;
        user-select: none;
        position: relative;
        perspective: 1000px;
        transform-style: preserve-3d;
        height: 100%;
        width: 100%;
    }

    .card .view {
        display: flex;
        align-items: center;
        justify-content: center;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        position: absolute;
        border-radius: 7px;
        background: #fff;
        pointer-events: none;
        backface-visibility: hidden;
        transition: transform 0.25s linear;
        border: 1px solid rgba(0, 0, 0, 0.096);
    }

    .card .front-view svg {
        width: 30px;
    }

    .card .back-view img {
        width: 45px;
        height: 45px;
        max-width: 45px;
        max-height: 45px;
    }

    .card .back-view span {
        display: none;
    }

    .card .back-view {
        transform: rotateY(-180deg);
    }

    .card.flip .back-view {
        transform: rotateY(0);
    }

    .card.flip .front-view {
        transform: rotateY(180deg);
    }
</style>