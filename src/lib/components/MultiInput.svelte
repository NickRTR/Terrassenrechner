<script>
	let {
		tags = $bindable([]),
		addKeySet = [13, 188, 32],
		onlyUnique = true,
		placeholder = "",
		labelText = "",
		labelShow = false,
		convertToNumber = false
	} = $props();

	let inputValue = $state("");
	let inputElement = $state();

	function focusInput() {
		setTimeout(() => inputElement?.focus(), 0);
	}

	function addTag(value = inputValue.trim()) {
		if (!value) return;
		let processedValue = convertToNumber ? Number(value) : value;
		if (convertToNumber && isNaN(processedValue)) return;
		if (onlyUnique && tags.includes(processedValue)) {
			inputValue = "";
			return;
		}
		tags = [...tags, processedValue];
		inputValue = "";
	}

	function removeTag(index) {
		tags = tags.filter((_, i) => i !== index);
		focusInput();
	}

	function handleKeydown(event) {
		if (addKeySet.includes(event.keyCode) || addKeySet.includes(event.which)) {
			event.preventDefault();
			addTag();
			return;
		}
		if (event.key === "Backspace" && inputValue === "" && tags.length > 0) {
			event.preventDefault();
			removeTag(tags.length - 1);
		}
	}

	function handlePaste(event) {
		event.preventDefault();
		const pastedText = event.clipboardData.getData("text");
		const values = pastedText.split(/[,\s]+/).filter((v) => v.trim());
		values.forEach((value) => addTag(value.trim()));
	}

	function handleBlur() {
		if (inputValue.trim()) addTag();
	}
</script>

<div class="multi-input-container">
	{#if labelShow && labelText}
		<label for="multi-input" class="label">{labelText}</label>
	{/if}

	<div class="input-wrapper">
		{#each tags as tag, index}
			<span class="tag">
				{tag}
				<button
					type="button"
					class="tag-remove"
					onclick={() => removeTag(index)}
					aria-label="Remove {tag}"
				>
					x
				</button>
			</span>
		{/each}

		<input
			bind:this={inputElement}
			bind:value={inputValue}
			onkeydown={handleKeydown}
			onpaste={handlePaste}
			onblur={handleBlur}
			type="text"
			id="multi-input"
			class="input"
			{placeholder}
			autocomplete="off"
		/>
	</div>
</div>

<style>
	.input-wrapper {
		display: flex;
		flex-wrap: wrap;
		align-items: center;
		gap: 0.25rem;
		padding: 0.25rem;
		margin-top: 0.5rem;
		box-shadow: 0 0 0 1px black;
		border-radius: 2px;
		background: white;
		min-height: 1.8rem;
	}

	.input-wrapper:focus-within {
		box-shadow: 0 0 0 2px #015fcc;
	}

	.tag {
		font-size: 0.8rem;
		display: inline-flex;
		align-items: center;
		gap: 0.25rem;
		padding: 0.125rem 0.375rem;
		background: #e9ecef;
		border: 1px solid #ced4da;
		border-radius: 3px;
	}

	.tag-remove {
		background: none;
		border: none;
		cursor: pointer;
		color: #6c757d;
		font-weight: bold;
		padding: 0;
		margin-left: 0.125rem;
	}

	.input {
		border: none;
		outline: none;
		background: transparent;
		flex: 1;
		min-width: 120px;
		font-size: 14px;
		line-height: 1.5;
	}

	.input::placeholder {
		color: #6c757d;
	}
</style>
