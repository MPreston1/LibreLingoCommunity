<script lang="ts">
	import { onMount } from 'svelte';
	import hotkeys from 'hotkeys-js';
	import leven from 'leven';
	import ChallengePanel from './ChallengePanel.svelte';
	import Icon from 'components/Icon.svelte';
	import InputFieldWithVirtualKeyboard from './InputFieldWithVirtualKeyboard/InputFieldWithVirtualKeyboard.svelte';
	import { playAudio } from '$lib/sounds';
	import Button from 'components/DeprecatedButton.svelte';
	import Column from 'components/Column.svelte';
	import Columns from 'components/Columns.svelte';

	export let challenge;
	export let registerResult;
	export let resolveChallenge;
	export let languageCode;
	export let specialCharacters;
	export let skipChallenge;
	export let skipAllChallenges;
	export let skipAllVoice;

	let answer = '';
	let submitted = false;
	let correct = null;
	let spellingSuggestion = '';

	$: submitChallenge = () => {
		if (!answer) return;
		if (submitted) return;
		const form = challenge.answer;
		correct = false;

		if (
			leven(
				answer
					.toLowerCase()
					.replace(/^\s+|\s+$/g, '')
					.replace(/\s+/g, ' '),
				form.toLowerCase()
			) <= 1
		) {
			correct = true;
			spellingSuggestion =
				form
					.replace(/^\s+|\s+$/g, '')
					.replace(/\s+/g, ' ')
					.toLowerCase() === answer.toLowerCase()
					? ''
					: `Correct spelling: ${form}`;
		}

		registerResult(correct);
		submitted = true;
	};

	$: finishChallenge = () => {
		answer = null;
		submitted = false;
		resolveChallenge();
	};

	const playChallengeVoice = () => playAudio('voice', challenge.audio);

	onMount(() => {
		playChallengeVoice();
		hotkeys.unbind('enter');
		hotkeys('enter', () => {
			if (submitted) {
				finishChallenge();
			} else {
				submitChallenge();
			}
		});
	});
</script>

<form on:submit|preventDefault={submitChallenge}>
	<div class="section">
		<p class="is-size-1 is-size-2-tablet is-size-4-mobile has-text-centered">Type what you hear</p>
	</div>

	<Columns>
		<Column size="1">
			<Button size="large" style="primary" on:click={playChallengeVoice}>
				<Icon icon="volume-up" />
			</Button>
		</Column>
		<Column>
			<InputFieldWithVirtualKeyboard
				{specialCharacters}
				{languageCode}
				disabled={submitted}
				bind:value={answer}
			/>
		</Column>
	</Columns>

	{#if answer && !submitted}
		<ChallengePanel
			message=""
			buttonText="Submit"
			submit
			skipAction={skipChallenge}
			skipAllAction={skipAllChallenges}
			{skipAllVoice}
		/>
	{/if}

	{#if answer === '' && !submitted}
		<ChallengePanel
			message={null}
			buttonText={null}
			skipAction={skipChallenge}
			skipAllAction={skipAllChallenges}
			{skipAllVoice}
		/>
	{/if}

	{#if submitted}
		{#if !correct}
			<ChallengePanel
				message="Incorrect solution!"
				messageDetail={`Correct answer: ${challenge.answer}`}
				buttonText="Continue"
				incorrect
				buttonAction={finishChallenge}
			/>
		{/if}

		{#if correct}
			{#if !spellingSuggestion}
				<ChallengePanel
					message="Correct solution!"
					messageDetail={`Meaning: "${challenge.meaning}"`}
					buttonText="Continue"
					correct
					buttonAction={finishChallenge}
				/>
			{/if}

			{#if spellingSuggestion}
				<ChallengePanel
					message="You have a typo!"
					messageDetail={spellingSuggestion || `Meaning: "${challenge.meaning}"`}
					buttonText="Continue"
					typo
					buttonAction={finishChallenge}
				/>
			{/if}
		{/if}
	{/if}
</form>
