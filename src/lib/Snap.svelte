<script lang="ts">

    // Import images
    import image1 from './images/img-1.png';
    import image2 from './images/img-2.png';
    import image3 from './images/img-3.png';
    import image4 from './images/img-4.png';
    import image5 from './images/img-5.png';
    import image6 from './images/img-6.png';
    import image7 from './images/img-7.png';
    import image8 from './images/img-8.png';

    // Array of images
    const images = [image1, image2, image3, image4, image5, image6, image7, image8];

    // State variables
    let flipped: number[] = $state([]); // Tracks currently flipped cards
    let matched: number[] = $state([]); // Tracks matched cards
    let cards: number[] = $state(Array.from({ length: 16 }, (_, i) => i)); // Card indices
    let startTime: number | null = $state(null); // Start time
    let timer: number | null = $state(null); // Timer
    let timerInterval: null | number; // Timer interval

    // Shuffle the cards
    function shuffle() {
        cards = cards.sort(() => Math.random() - 0.5);
    }

    // Reset the game
    function reset() {
        flipped = [];
        matched = [];
        shuffle();
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

    // Initial shuffle
    shuffle();
</script>

<!-- Card Grid -->
<div class="snap">
    <div class="cards">

        <!-- Preload images -->
        {#each images as image}
            <img class="hide" src={image} alt="card-img" />
        {/each}
    
        {#each cards as card}
            <button
                data-update={timer} 
                class="card {flipped.includes(card) || matched.includes(card) ? 'flip' : ''}"
                onclick="{() => flip(card)}"
            >
                <!-- Front of the card -->
                <div class="view front-view">
                    <svg id="Group_222" data-name="Group 222" xmlns="http://www.w3.org/2000/svg" width="24.738" height="43.032" viewBox="0 0 24.738 43.032">
                        <path id="Path_26" data-name="Path 26" d="M16.8,25.309c1.744-3.148,5.1-5.005,7.044-7.791,2.061-2.922.906-8.38-4.937-8.38-3.828,0-5.707,2.9-6.5,5.3L6.54,11.969A12.818,12.818,0,0,1,18.884,3c5.322,0,8.969,2.423,10.826,5.458,1.585,2.6,2.514,7.474.068,11.1-2.718,4.009-5.322,5.232-6.727,7.814-.566,1.042-.793,1.721-.793,5.073H15.713C15.69,30.677,15.418,27.8,16.8,25.309ZM23.436,41.5a4.53,4.53,0,1,1-4.53-4.53A4.543,4.543,0,0,1,23.436,41.5Z" transform="translate(-6.54 -3)" fill="#000"/>
                    </svg>
                </div>
    
                <!-- Back of the card -->
                <div class="view back-view">
                    {#if flipped.includes(card) || matched.includes(card)}
                        <img src={images[card % 8]} alt="card-img" />
                    {:else}
                        <span>Nice try! If you know what you're doing, why not contribute? <a href="https://github.com/doodad-labs/svelte-games">GitHub</a></span>
                    {/if}
                </div>
            </button>
        {/each}
    </div>
    
    <div  class="controls">
        {#if timer}
            <span>Time: {timer.toFixed(2)}s</span>
        {:else}
            <span>Click on a card to flip it</span>
        {/if}
    </div>
</div>

<!-- Styles -->
<style>
    .hide {
        display: none;
    }

    .snap {
        width: 300px;
    }

    .controls {
        width: 100%;
        margin-top: 10px;
    }

    .cards {
        height: 300px;
        width: 300px;
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
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        transition: transform 0.25s linear;
        border: 1px solid rgba(0, 0, 0, 0.096);
    }

    .card .front-view svg {
        width: 19px;
    }

    .card .back-view img {
        max-width: 45px;
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