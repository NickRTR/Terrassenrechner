<script>
	import Tags from "svelte-tags-input";

	// let längsteSeite = 0;
	// let dielenBreite = 0;
	// let dielenAbstand = 0;
	// let breiteTerrasse = 0;
	// let winkel = 0;

	let längsteSeite = 600;
	let dielenBreite = 11;
	let dielenAbstand = 5;
	let breiteTerrasse = 360;
	let winkel = 25;

	let plankSizes = [];

	function length(x) {
		// Convert the angle to radians
		const angleInRadians = winkel * (Math.PI / 180);
		// Calculate the pitch using trigonometry
		const pitch = Math.tan(angleInRadians);

		console.log(pitch);

		return pitch * x + längsteSeite;
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

		const breite = dielenBreite + 0.02 * dielenAbstand;
		for (let i = 0; i < breiteTerrasse; i += breite) {
			result.planks = [...result.planks, Math.round(length(i))];

			for (let j = 0; j < plankSizes.length; j++) {
				if (Math.round(length(i)) <= plankSizes[j]) {
					result.categorized[plankSizes[j]] += 1;
					break;
				}
			}
		}

		console.log(result.categorized);
	}
</script>

<body>
	<h1>Terrassendielen Rechner</h1>
	<img src="/Skizze.svg" alt="Skizze" width="500px" />

	<form on:submit={calculate}>
		<h2>Eingabe</h2>
		<label for="längsteSeite">Längste Seite: </label>
		<input type="number" bind:value={längsteSeite} min="0" max="10000" /> cm
		<br />
		<label for="breite">Dielenbreite: </label>
		<input type="number" bind:value={dielenBreite} min="0" max="10000" /> cm
		<br />
		<label for="dielenabstand">Abstand zwischen Dielen: </label>
		<input type="number" bind:value={dielenAbstand} min="0" max="7" /> mm
		<br />
		<label for="breite">Breite Terrasse: </label>
		<input type="number" bind:value={breiteTerrasse} min="0" max="10000" /> cm
		<br />
		<label for="winkel">Winkel: </label>
		<input type="number" bind:value={winkel} min="-80" max="80" /> °
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
