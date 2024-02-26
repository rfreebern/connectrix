<script>
	import { confetti } from '@neoconfetti/svelte';

	import { reduced_motion } from './reduced-motion';

	$: won = false;

	export function dragging(event) {
		event.dataTransfer.setData("id", event.target.id);
		event.dataTransfer.setData("word", event.target.dataset.word);
	}

	export function dropped(event) {
		event.preventDefault();

		// Remove the "solved" class from previously solved tiles so they can reanimate.
		[...document.getElementsByClassName("solved")].forEach((t) => t.classList.remove("solved"));

		// Move the word from the original tile to the grid slot.
		const original = document.getElementById(event.dataTransfer.getData("id"));
		event.target.appendChild(original);

		// See if a set has been correctly completed.
		validate();

		// Unset hover background, because I dunno, race condition?
		[...document.getElementsByClassName("slot")].forEach((s) => s.style.background = "white");
	}

	export function dragover(event) {
		event.preventDefault();
		event.dataTransfer.dropEffect = "move";
	}

	export function dragenter(event) {
		event.preventDefault();
		event.target.style.background = "rgba(0, 0, 0, 0.1)";
	}

	export function dragleave(event) {
		event.preventDefault();
		event.target.style.background = "white";
	}

	export function noDrop(event) {
		event.stopPropagation();
	}

	function shuffle(arr) {
		let i = arr.length;
		let j, temp;
		while (--i > 0) {
			j = Math.floor(Math.random()*(i + 1));
			temp = arr[j];
			arr[j] = arr[i];
			arr[i] = temp;
		}
	}

	const rowSolution = [
		{
			items: ['ONE', 'TWO', 'THREE', 'FOUR'],
			category: "A",
			complete: "🟩",
		},
		{
			items: ['FIVE', 'SIX', 'SEVEN', 'EIGHT'],
			category: "B",
			complete: "🟦",
		},
		{
			items: ['NINE', 'TEN', 'ELEVEN', 'TWELVE'],
			category: "C",
			complete: "🟪",

		},
		{
			items: ['THIRTEEN', 'FOURTEEN', 'FIFTEEN', 'SIXTEEN'],
			category: "D",
			complete: "🟨",
		}
	];

	const colSolution = [
		{
			items: ['ONE', 'FIVE', 'NINE', 'THIRTEEN'],
			category: "E",
			complete: "♥",
		},
		{
			items: ['TWO', 'SIX', 'TEN', 'FOURTEEN'],
			category: "F",
			complete: "♣",
		},
		{
			items: ['THREE', 'SEVEN', 'ELEVEN', 'FIFTEEN'],
			category: "G",
			complete: "♦",
		},
		{
			items: ['FOUR', 'EIGHT', 'TWELVE', 'SIXTEEN'],
			category: "H",
			complete: "♠",
		}
	];

	const words = rowSolution.map((r) => r.items).reduce((acc, curr) => acc.concat(curr));
	shuffle(words);

	export function wordAt(row, col) {
		return words[row * 4 + col];
	}

	function validate() {
		const rows = [...document.getElementsByClassName("grid")[0].getElementsByClassName("row")];
		let words, cols = [];
		rows.forEach((row) => {
			words = [];
			const slots = [...row.getElementsByClassName("slot")];
			slots.forEach((slot) => {
				const tile = [...slot.getElementsByClassName("tile")][0];
				if (tile) words.push(tile.dataset.word);
				else words.push('-');
			});
			cols.push([...words]);
			words.sort();
			rowSolution.forEach((row) => {
				if (!row.solved) {
					row.items.sort();
					if (JSON.stringify(row.items) == JSON.stringify(words)) {
						row.solved = true;
						addCompleteMarker(words, row.complete);
						addSolvedCategory(row);
					}
				}
			});
		});

		[0, 1, 2, 3].forEach((i) => {
			words = [];
			[0, 1, 2, 3].forEach((j) => {
				words.push(cols[j][i]);
			});
			words.sort();
			colSolution.forEach((col) => {
				if (!col.solved) {
					col.items.sort();
					if (JSON.stringify(col.items) == JSON.stringify(words)) {
						col.solved = true;
						addCompleteMarker(words, col.complete);
						addSolvedCategory(col);
					}
				}
			});
		});
	}

	function addCompleteMarker(words, char) {
		const rows = [...document.getElementsByClassName("grid")[0].getElementsByClassName("row")];
		rows.forEach((row) => {
			const tiles = [...row.getElementsByClassName("tile")];
			tiles.forEach((tile) => {
				if (words.includes(tile.dataset.word)) {
					tile.innerText += ` ${char}`;
					tile.classList.add("solved");
				}
			});
		});
		won = allSolved();
	}

	function addSolvedCategory(cat) {
		const list = [...document.getElementsByClassName("solvedList")][0];
		const item = document.createElement("li");
		item.innerText = `${cat.complete}: ${cat.category}`;
		list.appendChild(item);
	}

	function allSolved() {
		let solved = true;
		return rowSolution.reduce((acc, curr) => acc && !!curr.solved, solved)
			&& colSolution.reduce((acc, curr) => acc && !!curr.solved, solved);
	}

</script>

<svelte:head>
	<title>Connectrix</title>
	<meta name="description" content="2D Connections" />
</svelte:head>

<h1 class="visually-hidden">Connectrix</h1>
<h2 class="visually-hidden">2D Connections</h2>

	<div class="grid">
		{#each Array.from(Array(4).keys()) as row (row)}
			<h2 class="visually-hidden">Row {row + 1}</h2>
			<div class="row">
				{#each Array.from(Array(4).keys()) as column (column)}
					<div class="slot"
						on:dragover={dragover}
						on:dragenter={dragenter}
						on:dragleave={dragleave}
						on:drop={dropped}>
					</div>
				{/each}
			</div>
		{/each}
	</div>

	<div class="tiles">
		{#each Array.from(Array(4).keys()) as row (row)}
			<div class="row">
				{#each Array.from(Array(4).keys()) as column (column)}
					<div class="slot"
						on:dragover={dragover}
						on:dragenter={dragenter}
						on:dragleave={dragleave}
						on:drop={dropped}>
						<div
							class="tile"
							class:solved={false}
							draggable={true}
							on:dragstart={dragging}
							on:dragover={noDrop}
							on:dragenter={noDrop}
							on:dragleave={noDrop}
							on:drop={noDrop}
							data-word={wordAt(row, column)}
							id={`r${row}c${column}`}>
							{wordAt(row, column)}<br/>
						</div>
					</div>
				{/each}
			</div>
		{/each}
	</div>

	<div class="categories">
		<ul class="solvedList">
		</ul>
	</div>

{#if won}
	<div
		style="position: absolute; left: 50%; top: 25%"
		use:confetti={{
			particleCount: $reduced_motion ? 0 : undefined,
			force: 0.7,
			stageWidth: window.innerWidth,
			stageHeight: window.innerHeight,
			colors: ['#ff3e00', '#40b3ff', '#676778']
		}}
	/>
{/if}

<style>
	.grid, .tiles, .categories {
		--width: min(100vw, 50vh, 420px);
		max-width: var(--width);
		align-self: center;
		justify-self: center;
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		justify-content: flex-start;
	}

	.grid {
		margin-bottom: 1rem;
	}

	.grid .row, .tiles .row {
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		grid-gap: 0.2rem;
		margin: 0 0 0.2rem 0;
	}

	@media (prefers-reduced-motion: no-preference) {
		.tile.solved {
			animation: wiggle 0.25s;
		}
	}

	.slot, .tile {
		aspect-ratio: 1;
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		text-align: center;
		box-sizing: border-box;
		text-transform: uppercase;
		border: none;
		font-size: calc(0.03 * var(--width));
		font-weight: bold;
		border-radius: 2px;
		background: white;
		margin: 0;
		color: rgba(0, 0, 0, 0.7);
		}

	.slot {
		transition: background 0.25s ease-in;
	}

	.tile {
		cursor: grab;
	}

	@keyframes wiggle {
		0% {
			transform: scale(1);
		}
		10% {
			transform: scale(0.9);
		}
		30% {
			transform: scale(1.2);
		}
		50% {
			transform: scale(1.4);
		}
		70% {
			transform: scale(1.2);
		}
		90% {
			transform: scale(0.9);
		}
		100% {
			transform: scale(1);
		}
	}
</style>
