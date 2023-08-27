<script>
	import autoselect from "svelte-autoselect";

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

		if (result.planks.length === 0) {
			result.planks = null;
		}
	}
</script>

<body>
	<h1>Terrassendielen Rechner</h1>
	<p class="emphasize">Terrassenrechner zur Berechnung von Terrassenschrägen</p>
	<img src="/Sketch.svg" alt="Skizze" width="500px" />
	<br />

	<form on:submit={calculate}>
		<h2>Eingabe</h2>
		<label for="longestSide">Längste Seite: </label>
		<input type="number" use:autoselect bind:value={longestSide} min="0" max="10000" /> cm
		<br />
		<label for="shortestSide">Kürzeste Seite: </label>
		<input type="number" use:autoselect bind:value={shortestSide} min="0" max="10000" /> cm
		<br />
		<label for="plankWidth">Dielenbreite: </label>
		<input type="number" use:autoselect bind:value={plankWidth} min="0" max="10000" /> cm
		<br />
		<label for="plankSpace">Abstand zwischen Dielen: </label>
		<input type="number" use:autoselect bind:value={plankSpace} min="0" max="7" /> mm
		<br />
		<label for="width">Terrassenbreite: </label>
		<input type="number" use:autoselect bind:value={terraceWidth} min="0" max="10000" /> cm
		<br /><br />
		<div class="tagsContainer">
			<Tags
				addKeys={[9, 32, 188]}
				bind:tags={plankSizes}
				onlyUnique={true}
				placeholder="z.B. 300, 400, 500"
				labelText={"Dielenlängen (mm):"}
				labelShow
			/>
		</div>
		<br />
		<button type="submit">Berechnen</button>
	</form>

	{#if result.planks === null}
		<p class="error">Berechnung nicht möglich, bitte überprüfen Sie Ihre Eingaben.</p>
	{:else if result.planks.length > 0}
		<hr />
		<div class="ausgabe">
			<h2>Ausgabe</h2>
			<p>Anzahl Dielen: {result.planks.length}</p>

			{#if plankSizes.length > 0}
				<h3>Aufteilung:</h3>
				{#each Object.keys(result.categorized) as key}
					<p>
						{key} mm: {result.categorized[key]} Diele(n)
					</p>
				{/each}
			{/if}

			<h3>Auflistung:</h3>
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

<style>
	@font-face {
		font-family: "SourceCodePro";
		src: url("/fonts/SourceCodePro.ttf") format("truetype");
		font-display: swap;
	}

	* {
		font-family: "SourceCodePro";
	}

	.emphasize {
		text-decoration: dotted underline;
		text-underline-offset: 7px;
	}

	.tagsContainer {
		width: 400px;
	}

	button {
		font-weight: bold;
		margin-bottom: 1.5rem;
	}

	table,
	td,
	th {
		border-collapse: collapse;
		border: 2px solid black;
		border-collapse: collapse;
		padding: 0.3rem;
	}

	th {
		font-size: 0.9rem;
	}

	.error {
		color: red;
	}
</style>
