# svelte-games

Small games designed for loading/wait screens that can just be imported as a component with the ability to customise the experience.

## Install

```
npm i svelte-games
```

## Usage

```html
<script lang="ts">
    import { TicTacToe } from 'svelte-games';
</script>

<TicTacToe />
```

## Games

Name | Description | Import
--- | --- | ---
Tic Tac Toe | A classic strategy game where you face off against a bot in a 3x3 grid. Take turns marking spaces, aiming to be the first to get three of your marks in a row horizontally, vertically, or diagonally. | `svelte-games/TicTacToe.svelte`
Snap | A fast-paced matching game set on a 4x4 grid. Race against the clock to find and match all pairs of cards before time runs out. | `svelte-games/Snap.svelte`

## License

This project is licensed under the GPL-3.0 [License](LICENSE). See the LICENSE file for details.