<script lang="ts">
	import { tick } from 'svelte';

	let isrcInput = '';
	let isLoading = false;
	let resultUrl: string | null = null;
	let errorMessage: string | null = null;
	let successMessage: string | null = null;
	let formElement: HTMLFormElement;

	async function handleSubmit() {
		isLoading = true;
		resultUrl = null;
		errorMessage = null;
		successMessage = null;
		await tick(); // Ensure UI updates before fetch

		// Basic validation: split by newline, trim whitespace, filter empty lines
		const isrcs = isrcInput
			.split(/\r?\n/) // Split by newlines (Windows or Unix)
			.map(line => line.trim()) // Remove leading/trailing whitespace
			.filter(line => line.length > 0); // Remove empty lines

		if (isrcs.length === 0) {
			errorMessage = 'Please enter at least one ISRC.';
			isLoading = false;
			return;
		}

		try {
			const response = await fetch('/', { // POST to the current route's server endpoint
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify(isrcs) // Send the cleaned array
			});

			const data = await response.json();

			if (!response.ok || !data.success) {
				errorMessage = data.message || `Request failed with status ${response.status}`;
				// If there's a partial success URL, show it even on error
				if (data.url) {
					resultUrl = data.url;
				}
			} else {
				successMessage = data.message || 'Success!';
				resultUrl = data.url;
				isrcInput = ''; // Clear input on success
			}
		} catch (error: any) {
			console.error('Fetch error:', error);
			errorMessage = error.message || 'An unexpected error occurred.';
		} finally {
			isLoading = false;
		}
	}
</script>

<svelte:head>
	<title>ISRC to DAB Library</title>
	<meta name="description" content="Convert a list of ISRCs into a DAB library." />
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com">
	<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" rel="stylesheet">
</svelte:head>

<div class="container">
	<h1>ISRC to DAB Library Creator</h1>
	<p>Paste your ISRCs below (one per line) and click Create Library.</p>

	<form method="POST" on:submit|preventDefault={handleSubmit} bind:this={formElement}>
		<textarea
			bind:value={isrcInput}
			rows="10"
			placeholder="USRC12345678
USRC87654321
..."
			aria-label="ISRC Input"
			disabled={isLoading}
		></textarea>
		<button type="submit" disabled={isLoading}>
			{#if isLoading}
				<span class="spinner" aria-hidden="true"></span> Creating...
			{:else}
				Create Library
			{/if}
		</button>
	</form>

	{#if isLoading}
		<p class="status loading" aria-live="polite">Processing your request...</p>
	{/if}

	{#if successMessage}
		<p class="status success" aria-live="polite">{successMessage}</p>
	{/if}

	{#if errorMessage}
		<p class="status error" aria-live="polite">Error: {errorMessage}</p>
	{/if}

	{#if resultUrl}
		<p class="result">
			View your library: <a href={resultUrl} target="_blank" rel="noopener noreferrer">{resultUrl}</a>
		</p>
	{/if}

</div>

<style>
	/* CSS Variables for Theming */
	:root {
		--bg-color: #f9f9f9;
		--text-color: #333;
		--secondary-text-color: #555;
		--container-bg: #ffffff;
		--container-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
		--input-bg: #ffffff;
		--input-border: #ccc;
		--input-text: #333;
		--input-disabled-bg: #eee;
		--button-bg: #007bff;
		--button-text: #ffffff;
		--button-hover-bg: #0056b3;
		--button-disabled-bg: #aaa;
		--link-color: #007bff;
		--success-bg: #d4edda;
		--success-text: #155724;
		--success-border: #c3e6cb;
		--error-bg: #f8d7da;
		--error-text: #721c24;
		--error-border: #f5c6cb;
		--loading-bg: #e0e0e0;
		--loading-text: #555;
		--spinner-border: rgba(0, 0, 0, 0.1);
		--spinner-border-top: #555;
		--spinner-button-border: rgba(255, 255, 255, 0.3);
		--spinner-button-border-top: #fff;

		font-family: 'Inter', sans-serif;
	}

	:global(body) {
		background-color: var(--bg-color);
		color: var(--text-color);
		margin: 0;
		transition: background-color 0.3s, color 0.3s;
	}

	/* Dark Mode Styles */
	@media (prefers-color-scheme: dark) {
		:root {
			--bg-color: #1a1a1a;
			--text-color: #e0e0e0;
			--secondary-text-color: #aaaaaa;
			--container-bg: #2c2c2c;
			--container-shadow: 0 2px 10px rgba(0, 0, 0, 0.5);
			--input-bg: #3a3a3a;
			--input-border: #555;
			--input-text: #e0e0e0;
			--input-disabled-bg: #444;
			--button-bg: #0d6efd; /* Slightly brighter blue for dark mode */
			--button-text: #ffffff;
			--button-hover-bg: #0b5ed7;
			--button-disabled-bg: #555;
			--link-color: #60a5fa; /* Lighter blue for links */
			--success-bg: #1c4b2c; /* Darker green */
			--success-text: #c3e6cb;
			--success-border: #2a683d;
			--error-bg: #58151c; /* Darker red */
			--error-text: #f5c6cb;
			--error-border: #842029;
			--loading-bg: #444;
			--loading-text: #ccc;
			--spinner-border: rgba(255, 255, 255, 0.1);
			--spinner-border-top: #ccc;
		}
	}

	.container {
		max-width: 600px;
		margin: 2rem auto;
		padding: 1.5rem;
		background-color: var(--container-bg);
		border-radius: 8px;
		box-shadow: var(--container-shadow);
		text-align: center;
		transition: background-color 0.3s;
	}

	h1 {
		color: var(--text-color);
		margin-bottom: 1rem;
	}

	p {
		color: var(--secondary-text-color);
		line-height: 1.6;
	}

	form {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		margin-top: 1.5rem;
	}

	textarea {
		width: 100%;
		padding: 0.8rem;
		border: 1px solid var(--input-border);
		background-color: var(--input-bg);
		color: var(--input-text);
		border-radius: 4px;
		font-size: 1rem;
		font-family: monospace; /* Keep monospace for ISRCs */
		box-sizing: border-box;
		resize: vertical;
		transition: background-color 0.3s, border-color 0.3s, color 0.3s;
	}

	textarea:disabled {
		background-color: var(--input-disabled-bg);
		cursor: not-allowed;
	}

	button {
		padding: 0.8rem 1.5rem;
		background-color: var(--button-bg);
		color: var(--button-text);
		border: none;
		border-radius: 4px;
		font-size: 1rem;
		font-family: 'Inter', sans-serif; /* Use Inter for button */
		font-weight: bold;
		cursor: pointer;
		transition: background-color 0.2s ease;
		display: inline-flex;
		align-items: center;
		justify-content: center;
		gap: 0.5rem;
	}

	button:hover:not(:disabled) {
		background-color: var(--button-hover-bg);
	}

	button:disabled {
		background-color: var(--button-disabled-bg);
		cursor: not-allowed;
	}

	.status {
		margin-top: 1.5rem;
		padding: 0.8rem;
		border-radius: 4px;
		font-weight: bold;
		transition: background-color 0.3s, color 0.3s, border-color 0.3s;
	}

	.loading {
		background-color: var(--loading-bg);
		color: var(--loading-text);
	}

	.success {
		background-color: var(--success-bg);
		color: var(--success-text);
		border: 1px solid var(--success-border);
	}

	.error {
		background-color: var(--error-bg);
		color: var(--error-text);
		border: 1px solid var(--error-border);
	}

	.result {
		margin-top: 1rem;
		font-size: 1.1rem;
		word-wrap: break-word;
	}

	.result a {
		color: var(--link-color);
		text-decoration: none;
	}

	.result a:hover {
		text-decoration: underline;
	}

	/* Simple CSS spinner */
	.spinner {
		display: inline-block;
		width: 1em;
		height: 1em;
		border: 2px solid var(--spinner-button-border);
		border-radius: 50%;
		border-top-color: var(--spinner-button-border-top);
		animation: spin 1s ease-infinite;
	}

	@keyframes spin {
		to { transform: rotate(360deg); }
	}

</style>
