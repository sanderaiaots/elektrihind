<script>
	//https://dashboard.elering.ee/api/nps/price?start=2023-02-12T22:00:00.000Z&end=2023-02-13T21:59:59.999Z

	import { onMount } from 'svelte';
	import { apiData } from './hind.js';
	import { DateInput } from 'date-picker-svelte';
	let fromDate = new Date(new Date().getFullYear(), new Date().getMonth(), 1);
	let toDate = new Date();
	//new Date(currentDate);

	let cons = { '1675980000': 1 };
	let price = {};
	let sumCount = 0;
	let sumPrice = 0.0;
	let sumConsumed = 0.0;

	async function loadNps() {
		const editedTo = new Date(toDate);
		editedTo.setDate(editedTo.getDate() + 1);
		editedTo.setSeconds(editedTo.getSeconds() - 1);
		const url =
			'https://dashboard.elering.ee/api/nps/price?start=' +
			fromDate.toISOString() +
			'&end=' +
			editedTo.toISOString();
		fetch(url)
			.then((r) => r.json())
			.then((d) => {
				apiData.set(d.data.ee);
				price = {};
				d.data.ee.forEach((p) => {
					price[p.timestamp] = p.price;
				});
				parseFile();
			})
			.catch((error) => console.log('data error', error));
	}

	onMount(loadNps);
	let csvData = null;
	function parseTarbimine(event) {
		console.log('read data', event);
		if (event.target.files.length > 0) {
			const file = event.target.files[0];
			console.log('we got the file', file);
			const reader = new FileReader();
			reader.onload = fileReaded;
			reader.readAsText(file);
		}
	}

	function fileReaded(event) {
		csvData = event.target.result;
		parseFile();
	}

	function parseFile() {
		if (!csvData) {
			console.log('csv is empty', csvData);
			return;
		}
		//console.log(csvData);
		const lines = csvData.split('\n');
		let isAlgus = false;
		let tarbimine = {};
		sumPrice = 0.0;
		sumCount = 0;
		sumConsumed = 0.0;

		for (let i = 0; i < lines.length; i++) {
			const line = lines[i];

			if (isAlgus) {
				//console.log(line);
				const tokens = line.split(';');
				if (tokens && tokens.length && tokens.length > 3) {
					//console.log(line, new Date(tokens[0]));
					const dateHour = tokens[0].split(' ');
					const dateTokens = dateHour[0].split('.');
					const timeTokens = dateHour[1].split(':');

					const date = new Date(
						dateTokens[2],
						dateTokens[1] - 1,
						dateTokens[0],
						timeTokens[0],
						timeTokens[1]
					);
					/*
					console.log(
						'create date from ',
						dateTokens[2],
						dateTokens[1],
						dateTokens[0],
						timeTokens[0],
						timeTokens[1],
						date
					);*/

					const val = date.valueOf() / 1000;
					if (val == 1675638000) {
						console.log('see peab olemas olemas.....', line);
					}
					const key = val.toString();
					const consumedInHour = parseFloat(tokens[4].replace(',', '.'));
					tarbimine[key] = consumedInHour;
					if (price[key]) {
						sumPrice += (price[key] / 1000.0) * consumedInHour;
						sumConsumed += consumedInHour;
						sumCount++;
					}
				}
			}
			if (line.indexOf('Algus') >= 0) {
				isAlgus = true;
			}
		}
		//console.log(tarbimine);
		cons = tarbimine;
		//consumption.set(tarbimine);
		console.log('loaded ' + Object.keys(tarbimine).length + ' rows');
	}
</script>

<section>
	<div>Tere</div>
	<p>
		Lae oma tarbimise fail: <input type="file" id="consumtion-file" on:change={parseTarbimine} />
	</p>
	<div class="date">
		Alates: <DateInput bind:value={fromDate} format="dd.MM.yyyy" />
		Kuni: <DateInput bind:value={toDate} format="dd.MM.yyyy" />
		<button on:click={loadNps} value="Load NPS">Load NPS</button>
	</div>
	<p>
		Total consumed: {sumConsumed} kwH, hours {sumCount}, days {sumCount / 24} <br />
		Total price (no vat): {sumPrice} EUR, kwh avg {sumPrice / sumConsumed} EUR <br />
		Total price (vat): {sumPrice * 1.2} EUR, kwh avg {(sumPrice * 1.2) / sumConsumed} EUR
	</p>
	<table border="1" cellpadding="0" cellspacing="0">
		<thead>
			<tr><th>Tund</th><th>Hind</th><th>Hind + km</th><th>consumed</th><th>Hind kokku</th> </tr>
		</thead>
		<tbody>
			{#each $apiData as r}
				<tr>
					<td>{new Date(r.timestamp * 1000).toLocaleString()}</td>
					<td>{r.price}</td><td>{(r.price * 1.2).toFixed(2)}</td>
					<td>{cons[r.timestamp]}</td>
					<td>{(r.price * cons[r.timestamp]) / 1000.0}</td>
				</tr>
			{/each}
		</tbody>
	</table>
</section>

<style>
	.date {
		display: flex;
		align-items: center;
	}
</style>
