<script>
	import {
		Heading,
		P,
		Dropzone,
		Button,
		Footer,
		FooterCopyright,
		FooterIcon
	} from 'flowbite-svelte';
	import { GithubSolid, InstagramSolid, LinkedinSolid } from 'flowbite-svelte-icons';

	let selectedFile;
	const dropHandle = (event) => {
		event.preventDefault();

		let file;
		if (event.dataTransfer.items) {
			if (event.dataTransfer.items.length > 1) {
				return alert('Please upload only one file');
			}
			const item = event.dataTransfer.items[0];
			if (item.kind === 'file') {
				file = item.getAsFile();
			}
		} else {
			if (event.dataTransfer.files.length > 1) {
				return alert('Please upload only one file');
			}
			file = event.dataTransfer.files[0];
		}

		if (file) {
			if (!file.name.endsWith('.csv')) {
				return alert('Only .csv files are allowed');
			}
			selectedFile = file;
		}
	};

	const handleChange = (event) => {
		const files = event.target.files;
		if (files.length > 0) {
			if (files.length > 1) {
				return alert('Please upload only one file');
			}
			selectedFile = files[0];
		}
	};

	const cleanupFile = (text) => {
		// Split into lines and parse CSV
		const lines = text.split('\n');
		if (lines.length < 2) return text; // Return if empty or just header

		const header = lines[0];
		const headerFields = header.split(';');
		
		// Find required column indexes
		const contractIdIndex = headerFields.findIndex(field => field === 'Contract_ID');
		const costIndex = headerFields.findIndex(field => field === 'Calculated_Cost');
		
		if (contractIdIndex === -1 || costIndex === -1) {
			return text; // Return original if required columns not found
		}

		// Filter rows
		const filteredLines = lines.filter((line, index) => {
			if (index === 0) return true; // Keep header row
			
			const fields = line.split(';');
			if (fields.length !== headerFields.length) return false;
			
			const contractId = fields[contractIdIndex].trim();
			const cost = fields[costIndex].trim().replace(',', '.'); // Convert to decimal point
			
			return contractId !== '' && parseFloat(cost) !== 0;
		});

		// Rejoin filtered lines
		return filteredLines.join('\n');
		return text;
	};

	const convert = () => {
		const reader = new FileReader();
		reader.onload = (e) => {
			let content = e.target.result;
			content = cleanupFile(content);

			const blob = new Blob([content], { type: 'text/markdown' });
			const url = URL.createObjectURL(blob);
			const a = document.createElement('a');
			a.href = url;
			a.download = selectedFile.name.replace('.csv', '_clean.csv');
			document.body.appendChild(a);
			a.click();
			document.body.removeChild(a);
			URL.revokeObjectURL(url);
		};

		reader.readAsText(selectedFile);
	};
</script>

<svelte:head>
  <title>VTT to Markdown</title>
</svelte:head>

<div class="mx-auto mt-8 w-1/2 text-center">
	<Heading tag="h1" class="mb-4" customSize="text-4xl font-extrabold md:text-5xl lg:text-6xl"
		>📝 Longship CDR cleanup</Heading
	>
	<P class="mb-6 text-center text-lg dark:text-gray-400 lg:text-xl">
		This small tool allows you to upload a <code>.csv</code> CDR export from Longship and filter out
		ones with zero revenue and no Contract ID attached.
	</P>

	<Dropzone
		id="dropzone"
		on:drop={dropHandle}
		on:dragover={(event) => {
			event.preventDefault();
		}}
		on:change={handleChange}
	>
		<svg
			aria-hidden="true"
			class="mb-3 h-10 w-10 text-gray-400"
			fill="none"
			stroke="currentColor"
			viewBox="0 0 24 24"
			xmlns="http://www.w3.org/2000/svg"
			><path
				stroke-linecap="round"
				stroke-linejoin="round"
				stroke-width="2"
				d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"
			/></svg
		>
		{#if !selectedFile}
			<p class="mb-2 text-sm text-gray-500 dark:text-gray-400">
				<span class="font-semibold">Click to upload</span> or drag and drop
			</p>
			<p class="text-xs text-gray-500 dark:text-gray-400">.CSV file</p>
		{:else}
			<p>{selectedFile.name}</p>
		{/if}
	</Dropzone>

	<Button color="dark" size="xl" class="mt-8" on:click={convert} disabled={!selectedFile}
		>Cleanup</Button
	>

	<Footer>
		<div class="mt-16 sm:flex sm:items-center sm:justify-between">
			<FooterCopyright href="https://gaiacharge.com/" by="Gaia Charge" year={2025} />
			<div class="mt-4 flex space-x-6 sm:mt-0 sm:justify-center rtl:space-x-reverse">
				<FooterIcon href="https://www.instagram.com/gaia.charge/">
					<InstagramSolid
						class="h-5 w-5 text-gray-500 hover:text-gray-900 dark:text-gray-500 dark:hover:text-white"
					/>
				</FooterIcon>
				<FooterIcon href="https://www.linkedin.com/in/gaia-charge/">
					<LinkedinSolid
						class="h-5 w-5 text-gray-500 hover:text-gray-900 dark:text-gray-500 dark:hover:text-white"
					/>
				</FooterIcon>
				<FooterIcon href="https://github.com/gaia-charge">
					<GithubSolid
						class="h-5 w-5 text-gray-500 hover:text-gray-900 dark:text-gray-500 dark:hover:text-white"
					/>
				</FooterIcon>
			</div>
		</div>
	</Footer>
</div>
