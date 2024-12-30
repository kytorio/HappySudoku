<script>
	import Candidates from './Candidates.svelte';
	import { fade } from 'svelte/transition';
	import { SUDOKU_SIZE } from '@sudoku/constants';
	import { cursor } from '@sudoku/stores/cursor';
	import { grid, userGrid, referenceGrid, strategyGrid, invalidCells, strategyContent } from '@sudoku/stores/grid';
	import { candidates as candidatesStore } from '@sudoku/stores/candidates';
	import { BackupData } from '@sudoku/stores/data';

	export let value;
	export let cellX;
	export let cellY;
	export let candidates = [];

	export let disabled;
	export let conflictingNumber;
	export let userNumber;
	export let selected;
	export let sameArea;
	export let sameNumber;
	export let referenceNumber;

	const borderRight = (cellX !== SUDOKU_SIZE && cellX % 3 !== 0);
	const borderRightBold = (cellX !== SUDOKU_SIZE && cellX % 3 === 0);
	const borderBottom = (cellY !== SUDOKU_SIZE && cellY % 3 !== 0);
	const borderBottomBold = (cellY !== SUDOKU_SIZE && cellY % 3 === 0);

	function writeHint(pos) {
		// console.log(candidates);
		// console.log(pos.x, pos.y);
		if (candidates.length == 1) {
			userGrid.set(pos, candidates[0]);
			BackupData.add(pos,candidates[0]);
			cursor.reset();
			candidates = [];
			candidatesStore.clear(pos);
			strategyContent.clear();
			// console.log($userGrid);
		}
	}

	function getStrategy(pos) {
		// console.log($strategyContent);
		// console.log(pos.x, pos.y);
		cursor.set(pos.x, pos.y);
		// console.log($strategyGrid);
		let strategy = $strategyGrid[pos.x][pos.y];
		// console.log(strategy);
		strategyContent.set(strategy);
		// console.log($strategyContent);
	}

</script>

<div class="cell row-start-{cellY} col-start-{cellX}"
     class:border-r={borderRight}
     class:border-r-4={borderRightBold}
     class:border-b={borderBottom}
     class:border-b-4={borderBottomBold}>

	{#if !disabled}
		<div class="cell-inner"
			 class:user-number={userNumber}
			 class:selected={selected}
			 class:same-area={sameArea}
			 class:same-number={sameNumber}
			 class:conflicting-number={conflictingNumber}
			 class:hint-number={!selected && candidates && candidates.length > 0}
			 class:reference-number={referenceNumber}>
			 
			<button class="cell-btn" on:click={getStrategy({x: cellX - 1, y: cellY - 1})} on:dblclick={writeHint({x: cellX - 1, y: cellY - 1})}>
				{#if candidates && candidates.length > 1}
					<Candidates {candidates} />
				{:else if candidates && candidates.length === 1}
					<span class="cell-text">{candidates[0]}</span>
				{:else}
					<span class="cell-text">{value || ''}</span>
				{/if}
			</button>

		</div>
	{/if}

</div>

<style>
	.cell {
		@apply h-full w-full row-end-auto col-end-auto;
	}

	.cell-inner {
		@apply relative h-full w-full text-gray-800;
	}

	.cell-btn {
		@apply absolute inset-0 h-full w-full;
	}

	.cell-btn:focus {
		@apply outline-none;
	}

	.cell-text {
		@apply leading-full text-base;
	}

	@media (min-width: 300px) {
		.cell-text {
			@apply text-lg;
		}
	}

	@media (min-width: 350px) {
		.cell-text {
			@apply text-xl;
		}
	}

	@media (min-width: 400px) {
		.cell-text {
			@apply text-2xl;
		}
	}

	@media (min-width: 500px) {
		.cell-text {
			@apply text-3xl;
		}
	}

	@media (min-width: 600px) {
		.cell-text {
			@apply text-4xl;
		}
	}

	.user-number {
		@apply text-primary;
	}

	.selected {
		@apply bg-primary text-white;
	}

	.same-area {
		@apply bg-primary-lighter;
	}

	.same-number {
		@apply bg-primary-light;
	}

	.conflicting-number {
		@apply text-red-600;
	}

	.hint-number {
		@apply bg-green-400 text-white;
	}

	.reference-number {
		@apply bg-primary text-white;
	}
</style>