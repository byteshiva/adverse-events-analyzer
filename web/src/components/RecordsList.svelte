<div>
	<Paginator bind:pageNum {maxPageNum} />

	<div>
		<label>
			Page size
			<input type="number" value={pageSize} step="5" on:change={handlePageSizeChange} />
		</label>
	</div>

	<div class="table-container">
		<table>
			<thead>
				<tr>
					<th>Date</th>
					<th>MRN</th>
					<th>Diagnosis</th>
					<th>Procedure</th>
					<th>Anesthesiologist</th>
					<th>Complications</th>
					<th>Adverse events</th>
					<th>Anesthesia staff</th>
					<th>Location</th>
					<th>ASA</th>
					<th>Start</th>
					<th>Stop</th>
					<th>Smoker</th>
					<th>Age</th>
					<th>BMI</th>
				</tr>
			</thead>
			<tbody>
				{#each records as record}
					<tr>
						<td>
							<RichDate date={record.date} />
						</td>
						<td>{record.mrn}</td>
						<td>{record.diagnosis}</td>
						<td>{record.procedure}</td>
						<td>{record.anesthesiologist}</td>
						<td>
							{#if record.complications}
								Yes
							{:else if record.complications !== null}
								No
							{/if}
						</td>
						<td>
							<ul>
								{#each record.adverseEvents as event}
									<li>
										<a href="#" on:click={handleEventClick}>
											{event}
										</a>
									</li>
								{/each}
							</ul>
						</td>
						<td>
							<ul>
								{#each record.anesthesiaStaff as staffMember}
									<li>{staffMember}</li>
								{/each}
							</ul>
						</td>
						<td>{record.location}</td>
						<td>{record.asa}</td>
						<td>{record.anStart}</td>
						<td>{record.anStop}</td>
						<td>
							{#if record.smoker}
								✓
							{/if}
						</td>
						<td>{record.age}</td>
						<td>{record.bmi}</td>
					</tr>
				{/each}
			</tbody>
		</table>
	</div>

	<Paginator bind:pageNum {maxPageNum} />

	<div>
		<label>
			Page size
			<input type="number" value={pageSize} step="5" on:change={handlePageSizeChange} />
		</label>
	</div>
</div>

<script>
	import { getContext } from 'svelte';

	import RichDate from './RichDate.svelte';
	import Paginator from './Paginator.svelte';

	import { getRecords, len } from '../wasm-wrapper.js';

	export let viewHandle;

	let records = [];
	let numRecords = 0;
	let start;
	$: start = pageNum * pageSize;

	let pageNum = 0;
	let pageSize = 10;
	let maxPageNum;
	$: maxPageNum = Math.ceil(numRecords / pageSize) - 1;

	$: fetchRecords(viewHandle, start, pageSize);
	$: fetchLen(viewHandle);

	const { addEventFilter } = getContext('filter');

	async function handleEventClick(event) {
		event.preventDefault();

		const th = event.target;
		const eventName = th.textContent.trim();

		return addEventFilter(eventName);
	}

	async function fetchRecords(handle, start, length) {
		records = await getRecords(handle, start, length);
	}

	async function fetchLen(handle) {
		numRecords = await len(handle);
	}

	function handlePageSizeChange(event) {
		let value = Number(event.target.value);
		if (!Number.isNaN(value) && value > 0) {
			pageSize = value;
			pageNum = 0;
		}
	}
</script>

<style>
	ul {
		padding-left: 1em;
	}

	.table-container {
		width: 100%;
		overflow: auto;
	}
</style>
