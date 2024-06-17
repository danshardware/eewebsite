<script lang="ts">
    import { onMount } from 'svelte';

    export let title = "Resister";
    export let value = 10000.0
    export let resisterTolerance:"E6"|"E12"|"E24" = "E6"
    export let closestStandardResister = 10000.0

    let standardResistors = {
        "E6": [10.0, 15.0, 22.0, 33.0, 47.0, 68.0],
        "E12": [10.0, 12.0, 15.0, 18.0, 22.0, 27.0, 33.0, 39.0, 47.0, 56.0, 68.0, 82.0],
        "E24": [10.0, 11.0, 12.0, 13.0, 15.0, 16.0, 18.0, 20.0, 22.0, 24.0, 27.0, 30.0, 33.0, 36.0, 39.0, 43.0, 47.0, 51.0, 56.0, 62.0, 68.0, 75.0, 82.0, 91.0],
    } 
    let resistorSeries:number[] = []
    let tolerance = {
        "E6": 0.2,
        "E12": 0.1,
        "E24": 0.05,
    }

    function recalc() {
        resistorSeries.length = 0
        for(let decade = -1; decade < 9; decade++) {
            for(let mantisa of standardResistors[resisterTolerance]) {
                let value = (mantisa) * 10 ** decade
                resistorSeries.push(value)
            }
        }
        closestStandardResister = resistorSeries.reduce((prev, curr) => Math.abs(curr - value) < Math.abs(prev - value) ? curr : prev)
    }

    function humanReadableResistor(value:number): string {
        let unit = ""
        if(value >= 1e6) {
            value /= 1e6
            unit = "M"
        } else if(value >= 1e3) {
            value /= 1e3
            unit = "K"
        }
        return `${value.toLocaleString()}${unit}Ω`
    }

    onMount(() => {
        recalc()
    })
</script>

<div class="container">
    <section class="top-container">
        <h3>{title}</h3>
        <div class="form-row">
            <div class="input-data">
                <input type="number" id="resisterValue" bind:value={value} on:change={(e) => recalc()} />
                <div class="underline"></div>
                <label for="resisterValue">Resister Value</label>
            </div>
        </div>
        <div class="form-row">
            <div class="input-data">
				<select id="standardValue" bind:value={closestStandardResister} on:input={(e) => {if (e.target) value = e.target.value}}>
                    {#each resistorSeries as value}
                        <option value={value}>{humanReadableResistor(value)}</option>
                    {/each}
				</select>
				<div class="underline"></div>
				<label for="standardValue">Standard Value</label>
			</div>
        </div>
        <div class="form-row">
            <!-- Radio boxes to select the tolerance -->
            <div>
                <label for="E6">E6 (20%)</label>
                <input type="radio" id="E6" name="tolerance" value="E6" bind:group={resisterTolerance} on:change={(e) => recalc()} />
                &nbsp;
                <label for="E12">E12 (10%)</label>
                <input type="radio" id="E12" name="tolerance" value="E12" bind:group={resisterTolerance} on:change={(e) => recalc()} />
                &nbsp;
                <label for="E24">E24 (5%)</label>
                <input type="radio" id="E24" name="tolerance" value="E24" bind:group={resisterTolerance} on:change={(e) => recalc()} />
                &nbsp;
            </div>
        </div>
    </section>
</div>
<style>
	.container {
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 10px;
        border: var(--sidebar-color) 1px solid;
        width: 20rem;
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
	.input-data label {
		position: absolute;
		pointer-events: none;
		bottom: 10px;
		font-size: 16px;
		transition: all 0.3s ease;
		color: var(--secondary-color);
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