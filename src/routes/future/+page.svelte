<script>
	import { onMount } from 'svelte';
	import { apiData } from './data.js';

	let toDate = new Date();
	//new Date(currentDate);
	let networkPriceDay = 0.0369 * 100;
	let networkPriceNight = 0.021 * 100;
	let renewablePrice = 0.0124 * 100;
	let price = {};

	async function loadNps() {
		let baseDate = new Date();
		let fromDate = new Date(baseDate.getFullYear(), baseDate.getMonth(), baseDate.getDate());
		console.log('dates: ', baseDate.getFullYear(), baseDate.getMonth(), baseDate.getDate());
		console.log('parsed: ', fromDate);
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
					//price[p.timestamp] = p.price;
					p.jsdate = new Date(p.timestamp * 1000);
					const hour = p.jsdate.getHours();
					p.isDay = hour >= 7 && hour < 21;
				});
			})
			.catch((error) => console.log('data error', error));
	}

	onMount(loadNps);
</script>

<section>
	<p>
		Võrgutasu päev: <input type="number" value={networkPriceDay} />
		Võrgutasu öö: <input type="number" value={networkPriceNight} />
		Taastuvenergia tasu: <input type="number" value={renewablePrice} />
	</p>
	<p>Hinnad on senti kwh kohta</p>
	<table border="1" cellpadding="0" cellspacing="0">
		<thead>
			<tr>
				<th>Tund</th><th>Hind</th><th>Hind + km</th><th>Hind + võrk + taastuv</th>
			</tr>
		</thead>
		<tbody>
			{#each $apiData as r}
				<tr>
					<td
						>{r.jsdate.getDate()}.{r.jsdate.getMonth() + 1}.{r.jsdate.getFullYear()}
						{('' + r.jsdate.getHours()).padStart(2, '0')}-{(
							'' +
							(r.jsdate.getHours() + 1)
						).padStart(2, '0')}</td
					>
					<td style="text-align: right;">{(r.price / 10.0).toFixed(3)}</td>
					<td style="text-align: right;">{((r.price * 1.2) / 10.0).toFixed(3)}</td>
					<td style="text-align: right;"
						>{(
							(r.price / 10 + renewablePrice + (r.isDay ? networkPriceDay : networkPriceNight)) *
							1.2
						).toFixed(3)}</td
					>
				</tr>
			{/each}
		</tbody>
	</table>
</section>
