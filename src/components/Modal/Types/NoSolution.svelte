<script>
	import { difficulty as difficultyStore } from '@sudoku/stores/difficulty';
	import { startNew, startCustom } from '@sudoku/game';
	import { validateSencode } from '@sudoku/sencode';
	import { DIFFICULTIES } from '@sudoku/constants';
	import { userGrid, strategyGrid, referenceGrid, strategyContent } from '@sudoku/stores/grid';

	export let hideModal;

	//开始游戏 ，选择难度或输入sencode，sencode来自于游戏内生成的链接
	function handleContinue() {
		hideModal();
	}
</script>

<h1 class="text-3xl font-semibold mb-6 leading-none">No Solution</h1>

<label for="difficulty" class="text-lg mb-3">No Solution in current try. Go back find other way.</label>

<div class="flex justify-end">
	<button class="btn btn-small btn-primary" on:click={handleContinue}>Continue</button>
</div>