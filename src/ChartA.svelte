<script lang="ts">
	import { onMount } from 'svelte';
	import Chart from 'chart.js/auto';

	// ссылка на <canvas>
	let canvas: HTMLCanvasElement;

	onMount(() => {
		new Chart(canvas, {
			type: 'line',
			data: {
				labels: ['Jan', 'Feb', 'Mar', 'Apr'],
				datasets: [{
					label: 'Demo Data',
					data: [10, 20, 15, 25],
					borderWidth: 2
				}]
			},
			options: {
				responsive: true,
				maintainAspectRatio: false
			}
		});
	});
</script>

<canvas bind:this={canvas} style="width:100%; height:100%;"></canvas>
