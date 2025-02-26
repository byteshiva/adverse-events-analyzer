<div>
	<div class="trend-controls">
		<fieldset>
			<legend>Chart view</legend>

			<div class="view-selection-container">
				<div class="view-selection-group">
					<label>
						<input type="radio" bind:group={viewType} value={TimeseriesType.EventCount} />
						# with events
					</label>
					<label>
						<input type="radio" bind:group={viewType} value={TimeseriesType.EventPercentage} />
						% with events
					</label>
				</div>
				<div class="view-selection-group">
					<label>
						<input type="radio" bind:group={viewType} value={TimeseriesType.ComplicationSpecifiedCount} />
						# with complications specified
					</label>
					<label>
						<input type="radio" bind:group={viewType} value={TimeseriesType.ComplicationSpecifiedPercentage} />
						% with complications specified
					</label>
				</div>
				<div class="view-selection-group">
					<label>
						<input type="radio" bind:group={viewType} value={TimeseriesType.ComplicationOccurredCount} />
						# with complications occurred
					</label>
					<label>
						<input type="radio" bind:group={viewType} value={TimeseriesType.ComplicationOccurredPercentage} />
						% with complications occurred
					</label>
				</div>
			</div>
		</fieldset>
	</div>

	<Chart type="line" {data} title={getTitle(viewType)} {axisOptions} {lineOptions} bind:this={chart} />
</div>

<script lang="ts">
	import Chart from './FrappeChart.svelte';

	import { formatShortDate } from '../formatters.js';
	import { getTimeseries, Period, TimeseriesType } from '../wasm-wrapper.js';
	import type { DatePeriodNumber } from '../wasm-wrapper.js';

	export let viewHandle: number;

	let chart: Chart;

	let viewType = TimeseriesType.EventCount;

	let counts: DatePeriodNumber[] = [];
	$: getPeriodCounts(viewHandle, viewType);

	async function getPeriodCounts(handle: number, timeseriesType: TimeseriesType) {
		counts = await getTimeseries(handle, timeseriesType, Period.Day);
	}

	let labels: string[] = [];
	let values: number[] = [];

	$: labels = counts.map(count => formatShortDate(count.start));
	$: values = counts.map(count => count.value);

	let data = {};
	$: data = {
		labels,
		datasets: [
			{ values }
		],
	};

	const axisOptions = {
		xAxisMode: 'tick',
		yAxisMode: 'tick',
		xIsSeries: true,
	};

	const lineOptions = {
		hideDots: true
	};

	function getTitle(viewType: TimeseriesType) {
		switch (viewType) {
			case TimeseriesType.EventCount: return 'Number of events';
			case TimeseriesType.EventPercentage: return 'Percentage of events';
			case TimeseriesType.ComplicationSpecifiedCount: return 'Number of complication button presses';
			case TimeseriesType.ComplicationSpecifiedPercentage: return 'Percentage of cases with complication button presses';
			case TimeseriesType.ComplicationOccurredCount: return 'Number of cases with complications';
			case TimeseriesType.ComplicationOccurredPercentage: return 'Percentage of cases with complications';
		}
	}
</script>

<style>
	.trend-controls {
		display: flex;
		justify-content: flex-end;
	}

	.view-selection-container {
		display: flex;
		flex-wrap: wrap;
	}

	label {
		display: block;
		margin: 0.5em;
		white-space: nowrap;
	}
</style>
