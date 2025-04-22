<script lang="ts">
    import Chart from "chart.js/auto";
    import { onMount } from "svelte";

    let canvas: HTMLCanvasElement;
    onMount(() => {
        new Chart(canvas, {
			type: 'line',
			data: {
				labels: ['Jan', 'Feb', 'Mar', 'Apr'],
				datasets: [
					{
						label: 'Dataset 1',
						data: [10, 20, 15, 25],
						borderColor: 'rgb(75, 192, 192)',
						borderWidth: 2
					},
					{
						label: 'Dataset 2',
						data: [15, 10, 22, 18],
						borderColor: 'rgb(255, 99, 132)',
						borderWidth: 2
					},
					{
						label: 'Dataset 3',
						data: [5, 17, 13, 28],
						borderColor: 'rgb(54, 162, 235)',
						borderWidth: 2
					}
				]
			},
			options: {
				responsive: true,
				maintainAspectRatio: false
			}
		});
    });
</script>

<canvas bind:this={canvas} style="width:100%; height:100%;"></canvas>
