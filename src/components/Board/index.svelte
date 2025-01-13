<!--数独主界面的实现，仅为面板-->
<script>
	import { BOX_SIZE } from '@sudoku/constants';
	import { gamePaused } from '@sudoku/stores/game';
	import { grid, userGrid, referenceGrid, strategyGrid, invalidCells, strategyContent } from '@sudoku/stores/grid';
	import { settings } from '@sudoku/stores/settings';
	import { cursor } from '@sudoku/stores/cursor';
	import { candidates } from '@sudoku/stores/candidates';
	import Cell from './Cell.svelte';
	import Strategy from './Strategy.svelte';

	function isSelected(cursorStore, x, y) {	//标记选中的单元格
		return cursorStore.x === x && cursorStore.y === y;
	}

	function isSameArea(cursorStore, x, y) {	//标记相同区域，主要为行、列、BOX
		if (cursorStore.x === null && cursorStore.y === null) return false;
		if (cursorStore.x === x || cursorStore.y === y) return true;

		const cursorBoxX = Math.floor(cursorStore.x / BOX_SIZE);
		const cursorBoxY = Math.floor(cursorStore.y / BOX_SIZE);
		const cellBoxX = Math.floor(x / BOX_SIZE);
		const cellBoxY = Math.floor(y / BOX_SIZE);
		return (cursorBoxX === cellBoxX && cursorBoxY === cellBoxY);
	}

	function getValueAtCursor(gridStore, cursorStore) {	//获取光标所在的单元格的值，候选值视为0
		if (cursorStore.x === null && cursorStore.y === null) return null;

		return gridStore[cursorStore.y][cursorStore.x];
	}

	function isHintSelected(cursorStore) {
		// console.log(cursorStore.x, cursorStore.y);
		let vis = true;
		// console.log($candidates);
		// console.log($candidates[cursorStore.x + ',' + cursorStore.y]);
		if (!$candidates[cursorStore.x + ',' + cursorStore.y]) {
			vis = false;
		}
		// console.log(vis);
		return vis;
		// return $candidates[cursorStore.x + ',' + cursorStore.y] !== undefined && $candidates[cursorStore.x + ',' + cursorStore.y].length > 0;
	}

	function isReferenced(cursorStore, x, y) {
		if (cursorStore.x === null && cursorStore.y === null) 
			return false;
		// console.log(cursorStore.x, cursorStore.y);
		// console.log($referenceGrid[cursorStore.x][cursorStore.y]);
		let references = $referenceGrid[cursorStore.x][cursorStore.y];
		// console.log(references);
		let vis = references.some(ref => ref[0] === y && ref[1] === x);
		return vis;
	}

	$: formattedStrategyContent = $strategyContent.length > 0 ? $strategyContent.join('->') : '';
</script>

<div class="board-padding relative z-10">
	<div class="max-w-xl relative">
		<div class="w-full" style="padding-top: 100%"></div>
	</div>
	<div class="board-padding absolute inset-0 flex justify-center">

		<div class="bg-white shadow-2xl rounded-xl overflow-hidden w-full h-full max-w-xl grid" class:bg-gray-200={$gamePaused}>

			{#each $userGrid as row, y}
				{#each row as value, x}
					<Cell {value}
					      cellY={y + 1}
					      cellX={x + 1}
					      candidates={$candidates[x + ',' + y]}
					      disabled={$gamePaused}
					      selected={isSelected($cursor, x, y)}
					      userNumber={$grid[y][x] === 0}
					      sameArea={$settings.highlightCells && !isSelected($cursor, x, y) && isSameArea($cursor, x, y)}
					      sameNumber={$settings.highlightSame && value && !isSelected($cursor, x, y) && getValueAtCursor($userGrid, $cursor) === value}
					      conflictingNumber={$settings.highlightConflicting && $grid[y][x] === 0 && $invalidCells.includes(x + ',' + y)} 
						  referenceNumber={isHintSelected($cursor) && isReferenced($cursor, x, y)}/>
				{/each}
			{/each}

		</div>

	</div>
</div>
<Strategy content={formattedStrategyContent}/>
<style>
	.board-padding {
		@apply px-4 pb-4;
	}
</style>