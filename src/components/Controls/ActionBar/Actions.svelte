<script>
	import { candidates } from '@sudoku/stores/candidates';
	import { userGrid, strategyGrid, referenceGrid, strategyContent } from '@sudoku/stores/grid';
	import { cursor } from '@sudoku/stores/cursor';
	import { hints, clickNum } from '@sudoku/stores/hints';
	import { notes } from '@sudoku/stores/notes';
	import { settings } from '@sudoku/stores/settings';
	import { gamePaused,gameBackWard,gameForward,gameRecall } from '@sudoku/stores/game';
	import { ReCallGame,BackupGame,ForwardGame } from '@sudoku/game'
	import { solveSudokuTest } from '@sudoku/sudoku';
  	import { BackupData } from '@sudoku/stores/data';

	// $: hintsAvailable = $hints > 0;
	// let clickNum = 0;
	$: level = $settings.minhintlevelateachstep;
	// console.log("level: ", level);
	// console.log("clickNum: ", clickNum);
	let maxLevel = 3;

	function handleReCallGame() {
		ReCallGame();
		reset(false);
		solve();
	}

	function handleBackupGame() {
		BackupGame();
		reset(true);
		solve();
	}

	function reset(same) {
		// console.log(clickNum);
		// console.log(same);
		// clickNum = 0;
		clickNum.update($clickNum => $clickNum = 0);
		$userGrid.forEach((row, rowIndex) => {
			row.forEach((cell, colIndex) => {
				candidates.clear({x: colIndex, y: rowIndex});	
			});
		});
		if (!same) {
			localStorage.setItem('userGrid', JSON.stringify($userGrid));
			$userGrid.forEach((row, rowIndex) => {
				row.forEach((cell, colIndex) => {
					// hintGrid.clear({x: rowIndex, y: colIndex});	
					strategyGrid.clear({x: rowIndex, y: colIndex});	
					referenceGrid.clear({x: rowIndex, y: colIndex});	
				});
				strategyContent.clear();
			});
		}
	}

	function solve() {
		// console.log(222);
		// console.log(clickNum);
		let [possibleNumbers, referenceNumbers, strategy] = solveSudokuTest($userGrid);
		// console.log(possibleNumbers);
		// console.log(referenceNumbers);
		// console.log(strategy);
		$userGrid.forEach((row, rowIndex) => {
			row.forEach((cell, colIndex) => {
				candidates.clear({x: colIndex, y: rowIndex});	
			});
		});
		let res = JSON.parse(JSON.stringify(possibleNumbers));
		res.forEach((row, rowIndex) => {
			row.forEach((element, colIndex) => {
				// console.log(rowIndex, colIndex);
				if (element.length > level + $clickNum) {
					res[rowIndex][colIndex] = [];
					referenceNumbers[rowIndex][colIndex] = [];
					strategy[rowIndex][colIndex] = "";
				} else {
					element.forEach(value => {
						candidates.add({ x: colIndex, y: rowIndex }, value);
					});
				}
			});
		});
		strategy.forEach((row, rowIndex) => {
			row.forEach((element, colIndex) => {
				strategyGrid.set({x: rowIndex, y: colIndex}, element);
			});
		});
		referenceNumbers.forEach((row, rowIndex) => {
			row.forEach((element, colIndex) => {
				referenceGrid.set({x: rowIndex, y: colIndex}, element);
			});
		});
		// clickNum += 1;
		clickNum.update($clickNum => $clickNum + 1);
	}

	function handleHint() {	//提示
		// console.log("level: ", level);
		// console.log("clickNum: ", $clickNum);
		// console.log(level, $clickNum, maxLevel);
		const storedUserGrid = localStorage.getItem('userGrid');
		let userGrid2 = JSON.parse(storedUserGrid);
		// console.log($userGrid);
		// console.log(userGrid2);
		let same = JSON.stringify($userGrid) === JSON.stringify(userGrid2)
		// console.log(same);
		if (level + $clickNum > maxLevel || !same) {
			reset(same);
		} else {
			solve();
		}
	}
</script>

<div class="action-buttons space-x-3">

	<button class="btn btn-round" on:click={handleReCallGame} disabled={!$gameRecall} title="Recall">	<!--TODO：回溯-->
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6a10 10 0 1 1 -2 4 m2 -4 l6 2m-6-2l2 -6" />
		</svg>
	</button>

	<button class="btn btn-round" on:click={handleBackupGame} disabled={!$gameBackWard} title="Undo">	<!--TODO：撤销-->
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 10h10a8 8 0 018 8v2M3 10l6 6m-6-6l6-6" />
		</svg>
	</button>


	<button class="btn btn-round"  on:click={ForwardGame} disabled={!$gameForward} title="Redo"> <!--TODO：前进-->
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 10h-10a8 8 90 00-8 8v2M21 10l-6 6m6-6l-6-6" />
		</svg>
	</button>

	<button class="btn btn-round btn-badge" on:click={handleHint} title="Hints ({$hints})">
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z" />
		</svg>

		<!-- {#if $settings.hintsLimited}
			<span class="badge" class:badge-primary={hintsAvailable}>{$hints}</span>
		{/if} -->
	</button>

	<button class="btn btn-round btn-badge" on:click={notes.toggle} title="Notes ({$notes ? 'ON' : 'OFF'})">
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" />
		</svg>

		<span class="badge tracking-tighter" class:badge-primary={$notes}>{$notes ? 'ON' : 'OFF'}</span>
	</button>

</div>


<style>
	.action-buttons {
		@apply flex flex-wrap justify-evenly self-end;
	}

	.btn-badge {
		@apply relative;
	}

	.badge {
		min-height: 20px;
		min-width:  20px;
		@apply p-1 rounded-full leading-none text-center text-xs text-white bg-gray-600 inline-block absolute top-0 left-0;
	}

	.badge-primary {
		@apply bg-primary;
	}
</style>