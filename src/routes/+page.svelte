<script>
	import Tags from "svelte-tags-input";

	let longestSide = 0;
	let shortestSide = 0;
	let plankWidth = 0;
	let plankSpace = 0;
	let terraceWidth = 0;

	let plankSizes = [];

	function length(x) {
		const pitch = (longestSide - shortestSide) / terraceWidth;
		return pitch * x + shortestSide;
	}

	let result = {
		planks: [],
		categorized: {}
	};

	function calculate() {
		result = {
			planks: [],
			categorized: {}
		};

		plankSizes = plankSizes.sort((a, b) => a - b);
		for (const plankSize of plankSizes) {
			result.categorized[plankSize] = 0;
		}

		const width = plankWidth + 0.02 * plankSpace;
		for (let i = 0; i < terraceWidth; i += width) {
			result.planks = [...result.planks, Math.round(length(i))];

			for (let j = 0; j < plankSizes.length; j++) {
				if (Math.round(length(i)) <= plankSizes[j]) {
					result.categorized[plankSizes[j]] += 1;
					break;
				}
			}
		}
	}
</script>

<body>
	<h1>Terrassendielen Rechner</h1>
	<img src="/Skizze.svg" alt="Skizze" width="500px" />
	<br />

	<form on:submit={calculate}>
		<h2>Eingabe</h2>
		<label for="longestSide">Längste Seite: </label>
		<input type="number" bind:value={longestSide} min="0" max="10000" /> cm
		<br />
		<label for="shortestSide">Kürzeste Seite: </label>
		<input type="number" bind:value={shortestSide} min="0" max="10000" /> cm
		<br />
		<label for="plankWidth">Dielenbreite: </label>
		<input type="number" bind:value={plankWidth} min="0" max="10000" /> cm
		<br />
		<label for="plankSpace">Abstand zwischen Dielen: </label>
		<input type="number" bind:value={plankSpace} min="0" max="7" /> mm
		<br />
		<label for="width">Breite Terrasse: </label>
		<input type="number" bind:value={terraceWidth} min="0" max="10000" /> cm
		<br /><br />
		<Tags
			addKeys={[9, 32, 188]}
			bind:tags={plankSizes}
			onlyUnique={true}
			placeholder="z.B. 300, 400, 500"
			labelText={"Dielenlängen (mm):"}
			labelShow
		/>
		<br />
		<button type="submit">Berechnen</button>
	</form>

	{#if result.planks.length > 0}
		<div class="ausgabe">
			<h2>Ausgabe</h2>
			<p>Anzahl Dielen: {result.planks.length}</p>

			<h3>Aufteilung:</h3>
			{#each Object.keys(result.categorized) as key}
				<p>
					{key} mm: {result.categorized[key]} Diele(n)
				</p>
			{/each}

			<table>
				<thead>
					<th>Diele Nr.</th>
					<th>Dielenlänge</th>
				</thead>
				{#each result.planks as plank, i}
					<tr>
						<td>{i + 1}</td>
						<td>{plank} cm</td>
					</tr>
				{/each}
			</table>
		</div>
	{/if}
</body>
