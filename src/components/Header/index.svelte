<!--顶部栏的实现，Dropdown为下拉框，Buttons为右侧按钮-->
<script>
	import Buttons from './Buttons.svelte';
	import Dropdown from './Dropdown.svelte';
</script>

<div class="px-4 py-4 flex justify-center text-white">
	<div class="w-full max-w-xl">

		<nav class="flex flex-wrap items-center justify-between">
			<Dropdown />

			<Buttons />
		</nav>

	</div>
</div>