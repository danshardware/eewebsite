<script lang="ts">
	import { Chart, type ChartData, type ChartItem, registerables } from 'chart.js';
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';

	export let R25 = 10000.0;
	export let beta = 3435.0;
	export let Vref = 3.3;
	export let Vadc = 3.3;
	export let R1 = 10000.0;
	export let R2 = 10000.0;
	export let Ir = 0.000001;
	export let tempMin = -40;
	export let tempMax = 125;
	export let topology: 'voltage-divider' | 'current-source' | 'Voltage-divider-parallel' =
		'voltage-divider';
	export let adcBits = 0;

	let barChartElement: HTMLCanvasElement;
	let chart: Chart;
	Chart.register(...registerables);

	const chartData: ChartData = {
		labels: [],
		datasets: [
			{
				data: [],
				fill: false
				// borderColor: 'rgb(75, 192, 192)',
				// tension: 0.1
			}
		]
	};

	function voltageToAdc(v: number) {
		let bits = Math.round((v / Vadc) * 2 ** adcBits);
		if (bits < 0) {
			return 0;
		}
		if (bits > 2 ** adcBits - 1) {
			return 2 ** adcBits - 1;
		}
		return bits;
	}

	function calculateCurves() {
		chartData.labels = [...Array.from({ length: tempMax - tempMin }, (x, i) => i + tempMin)];
		chartData.datasets[0].data = chartData.labels.map((temp) => {
			let t = Number(temp);
			return R25 * Math.exp(beta * (1 / (t + 273.15) - 1 / 298.15));
		});
		chartData.datasets[0].label = 'Resistance';
		chartData.datasets[1] = {
			data: [],
			fill: false,
			// borderColor: 'rgb(75, 192, 192)',
			// tension: 0.1,
			label: adcBits === 0 ? 'Voltage' : 'ADC Counts',
			yAxisID: 'y1'
		};
		switch (topology) {
			case 'voltage-divider':
				chartData.datasets[1].data = chartData.datasets[0].data.map((r) => {
					if (!r || !R1 || !Vref) {
						return 0;
					}
					let rr = Number(r);
					let v = Number(Vref) * (rr / (R1 + rr));
					if (adcBits) {
						return voltageToAdc(v);
					}
					return v;
				});
				break;
			case 'current-source':
				chartData.datasets[1].data = chartData.datasets[0].data.map((r) => {
					if (!r || !R1 || !Vref) {
						return 0;
					}
					let rr = Number(r);
					let v = rr * Ir;
					if (adcBits) {
						return voltageToAdc(v);
					}
					return v;
				});
				break;
			case 'Voltage-divider-parallel':
				chartData.datasets[1].data = chartData.datasets[0].data.map((r) => {
					if (!r || !R1 || !Vref) {
						return 0;
					}
					let rr = 1 / (1 / Number(r) + 1 / R2);
					let v = Number(Vref) * (rr / (R1 + rr));
					if (adcBits) {
						return voltageToAdc(v);
					}
					return v;
				});
				break;
		}
		chart.update();
	}

	onMount(() => {
		if (browser) {
			// barChartElement = document.createElement('canvas');
			chart = new Chart(barChartElement.getContext('2d') as ChartItem, {
				type: 'line',
				data: chartData,
				options: {
					responsive: true,
					plugins: {
						legend: {
							position: 'top'
						},
						title: {
							display: true,
							text: 'NTC Thermistor Curve'
						}
					},
					scales: {
						y: {
							type: 'linear',
							display: true,
							position: 'left'
						},
						y1: {
							type: 'linear',
							display: true,
							position: 'right',

							// grid line settings
							grid: {
								drawOnChartArea: false // only want the grid lines for one axis to show up
							}
						}
					}
				}
			});
		}
		calculateCurves();
	});
</script>

<h2>NTC Thermister Curve Prediction</h2>
<div class="container">
	<section class="top-container">
		<h3>NTC Parameters</h3>
		<div class="form-row">
			<div class="input-data">
				<input
					type="number"
					id="tempMin"
					bind:value={tempMin}
					on:change={(e) => calculateCurves()}
				/>
				<div class="underline"></div>
				<label for="tempMin">tempMin</label>
			</div>
			<div class="input-data">
				<input
					type="number"
					id="tempMax"
					bind:value={tempMax}
					on:change={(e) => calculateCurves()}
				/>
				<div class="underline"></div>
				<label for="tempMax">tempMax</label>
			</div>
		</div>
		<div class="form-row">
			<div class="input-data">
				<input type="number" id="R25" bind:value={R25} on:change={(e) => calculateCurves()} />
				<div class="underline"></div>
				<label for="R25">R25</label>
			</div>
			<div class="input-data">
				<input type="number" id="beta" bind:value={beta} on:change={(e) => calculateCurves()} />
				<div class="underline"></div>
				<label for="beta">Beta</label>
			</div>
		</div>
		<div class="form-row">
			<div class="input-data">
				<input type="number" min="1.0" max="100.0" step = ".1" id="Vref" bind:value={Vref} on:change={(e) => calculateCurves()} />
				<div class="underline"></div>
				<label for="Vref">Vref</label>
			</div>
			<div class="input-data">
				<select id="topology" bind:value={topology} on:change={(e) => calculateCurves()}>
					<option value="voltage-divider">Voltage Divider</option>
					<option value="current-source">Current Source</option>
					<option value="Voltage-divider-parallel">Voltage Divider Parallel</option>
				</select>
				<div class="underline"></div>
				<label for="topology">Topology</label>
			</div>
		</div>
		<div class="form-row">
			{#if topology === 'current-source'}
				<div class="input-data">
					<input type="number" min="0" max="1" step=".000001" id="Ir" bind:value={Ir} on:change={(e) => calculateCurves()} />
					<div class="underline"></div>
					<label for="Ir">Ir</label>
				</div>
			{/if}
			{#if topology !== 'current-source'}
				<div class="input-data">
					<input type="number" min="10" max="10000000" step="500" id="R1" bind:value={R1} on:change={(e) => calculateCurves()} />
					<div class="underline"></div>
					<label for="R1">R1</label>
				</div>
			{/if}
			{#if topology === 'Voltage-divider-parallel'}
				<div class="input-data">
					<input type="number" min="10" max="10000000" step="500" id="R2" bind:value={R2} on:change={(e) => calculateCurves()} />
					<div class="underline"></div>
					<label for="R2">R2</label>
				</div>
			{/if}
		</div>
		<div class="form-row">
			<div class="input-data">
				<input
					type="number"
					id="adcBits"
					min="0"
					max="32"
					bind:value={adcBits}
					on:change={(e) => calculateCurves()}
				/>
				<div class="underline"></div>
				<label for="adcBits">ADC Bits</label>
			</div>
			{#if adcBits !== 0}
			<div class="input-data">

				<input type="number" min="1.0" max="100.0" id="Vadc" step = ".1" bind:value={Vadc} on:change={(e) => calculateCurves()} />
				<div class="underline"></div>
				<label for="Vadc">Vadc</label>
			</div>
			{/if}
		</div>
	</section>
</div>
<section>
	<h3>NTC Curve</h3>
	<canvas bind:this={barChartElement} />
</section>

<style>
	.container {
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 10px;
	}
	.top-container {
		max-width: 800px;
		padding: 25px 40px 10px 40px;
	}

	.container .form-row {
		display: flex;
		margin: 32px 0;
	}
	.form-row .input-data {
		width: 100%;
		height: 40px;
		margin: 0 20px;
		position: relative;
	}
	.form-row .textarea {
		height: 70px;
	}
	.input-data input,
	.input-data select,
	.textarea textarea {
		display: block;
		width: 100%;
		height: 100%;
		border: none;
		font-size: 17px;
		border-bottom: 2px solid rgba(0, 0, 0, 0.12);
	}
	.input-data input:focus ~ label,
	.textarea textarea:focus ~ label,
	.input-data select:valid ~ label,
	.input-data input:valid ~ label,
	.input-data input:in-range ~ label,
	.textarea textarea:valid ~ label {
		transform: translateY(-20px);
		font-size: 14px;
		color: var(--secondary-color);
	}
	.textarea textarea {
		resize: none;
		padding-top: 10px;
	}
	.input-data label {
		position: absolute;
		pointer-events: none;
		bottom: 10px;
		font-size: 16px;
		transition: all 0.3s ease;
		color: var(--secondary-color);
	}
	.textarea label {
		width: 100%;
		bottom: 40px;
	}
	.input-data .underline {
		position: absolute;
		bottom: 0;
		height: 2px;
		width: 100%;
	}
	.input-data .underline:before {
		position: absolute;
		content: '';
		height: 2px;
		width: 100%;
		background: #3498db;
		transform: scaleX(0);
		transform-origin: center;
		transition: transform 0.3s ease;
	}
	.input-data input:focus ~ .underline:before,
	.input-data input:valid ~ .underline:before,
	.input-data input:in-range ~ .underline:before,
	.input-data select:valid ~ .underline:before,
	.textarea textarea:focus ~ .underline:before,
	.textarea textarea:valid ~ .underline:before {
		transform: scale(1);
	}
</style>
