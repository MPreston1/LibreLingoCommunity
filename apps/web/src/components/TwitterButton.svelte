<script lang="ts">
	import Button from 'components/DeprecatedButton.svelte';
	import Icon from 'components/Icon.svelte';
</script>

<Button
	target="_blank"
	href="https://twitter.com/intent/tweet?hashtags=LibreLingo%2Copensource&url=https%3A%2F%2Flibrelingo.app&text=LibreLingo%20-%20an%20experiment%20to%20create%20a%20community-owned%20language%20learning%20tool%0A"
>
	<slot name="icon-left">
		<Icon icon="twitter" prefix="fab" size="small" />
	</slot>
	<div>Tweet about LibreLingo</div>
</Button>
