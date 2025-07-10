<script lang="ts">
	import autoselect from "svelte-autoselect"
	import MultiInput from "$lib/components/MultiInput.svelte"

	let longestSide = $state(0)
	let shortestSide = $state(0)
	let plankWidth = $state(0)
	let plankSpace = $state(0)
	let terraceWidth = $state(0)

	let plankSizes = $state<number[]>([])

	function length(x: number) {
		const pitch = (longestSide - shortestSide) / terraceWidth
		return pitch * x + shortestSide
	}

	let result = $state<{
		planks: number[] | null
		categorized: Record<string, number>
	}>({
		planks: [],
		categorized: {}
	})

	function calculate() {
		result = {
			planks: [],
			categorized: {}
		}

		let categorizedPlanks = 0

		plankSizes = plankSizes.sort((a, b) => a - b)
		for (const plankSize of plankSizes) {
			result.categorized[plankSize + "cm"] = 0
		}

		const width = plankWidth + 0.05 * plankSpace
		for (let i = 0; i < terraceWidth; i += width) {
			result.planks = [...(result.planks as number[]), Math.round(length(i))]

			for (let j = 0; j < plankSizes.length; j++) {
				if (Math.round(length(i)) <= plankSizes[j]) {
					result.categorized[plankSizes[j] + "cm"] += 1
					categorizedPlanks++
					break
				}
			}
			result.categorized[plankSizes[plankSizes.length - 1] + "cm+"] += 1
		}

		result.categorized[plankSizes[plankSizes.length - 1] + "cm+"] =
			(result.planks as number[]).length - categorizedPlanks

		if ((result.planks as number[]).length === 0) {
			result.planks = null
		}
	}
</script>

<div>
	<h1>Terrassendielen Rechner</h1>
	<p class="emphasize">Terrassenrechner zur Berechnung von Terrassenschrägen</p>
	<img src="/Sketch.svg" alt="Skizze" width="500px" />
	<br />

	<form onsubmit={calculate}>
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
		<input type="number" use:autoselect bind:value={plankSpace} min="0" max="10" /> mm
		<br />
		<label for="width">Terrassenbreite: </label>
		<input type="number" use:autoselect bind:value={terraceWidth} min="0" max="10000" /> cm
		<br /><br />
		<div class="tagsContainer">
			<MultiInput
				bind:tags={plankSizes}
				placeholder="z.B. 300, 400, 500"
				labelText={"Dielenlängen (cm):"}
				labelShow
				convertToNumber={true}
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
					<p class:warning={key.includes("+")}>
						{key}: {result.categorized[key]} Diele(n)
					</p>
				{/each}
			{/if}

			<h3>Auflistung:</h3>
			<table>
				<thead>
					<tr>
						<th>Diele Nr.</th>
						<th>Dielenlänge</th>
					</tr>
				</thead>
				<tbody>
					{#each result.planks as plank, i}
						<tr>
							<td>{i + 1}</td>
							<td>{plank} cm</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	{/if}
</div>

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
		margin-top: 0.25rem;
		font-weight: bold;
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

	.warning {
		color: red;
		font-weight: 600;
	}
</style>
