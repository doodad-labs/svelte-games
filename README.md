# svelte-games

**Small, interactive games designed for loading or wait screens.** Easily import these lightweight, customizable game components into your Svelte projects to keep users entertained during downtime.

[🎮 Play some of the games now!](https://doodad-labs.github.io/svelte-games/)

## Quick Start

### Installation

Install the package via npm:

```bash
npm install svelte-games
```

### Usage

Simply import the game component and drop it into your Svelte project:

```svelte
<script lang="ts">
  import TicTacToe from 'svelte-games/TicTacToe.svelte';
</script>

<TicTacToe />
```

## Available Games

Game | Description | Import Path                     
--- | --- | ---
**Tic Tac Toe** | A classic strategy game where you face off against a bot on a 3x3 grid. Aim to get three of your marks in a row (horizontally, vertically, or diagonally) to win. | `svelte-games/TicTacToe.svelte`
**Snap** | A fast-paced matching game on a 4x4 grid. Race against the clock to find and match all pairs of cards before time runs out. | `svelte-games/Snap.svelte`
**Snake** | A classic arcade game where you control a growing snake. Navigate the snake to eat food and grow longer, but avoid colliding with the walls or yourself. | `svelte-games/Snake.svelte`
**Sudoku** | A challenging logic-based number puzzle. Fill a 9x9 grid so that each row, column, and 3x3 subgrid contains all digits from 1 to 9 without repetition. | `svelte-games/Sudoku.svelte`

## License

This project is licensed under the **GPL-3.0 License**. For more details, see the [LICENSE](LICENSE) file.

## Contributing

Contributions are welcome! If you have ideas for new games or improvements, feel free to open an issue or submit a pull request.

Enjoy adding a touch of fun to your Svelte projects!
*Built with ❤️ by [Doodad Labs](https://github.com/doodad-labs).*
