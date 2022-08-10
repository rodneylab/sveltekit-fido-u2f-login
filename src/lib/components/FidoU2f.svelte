<script>
	import { browser } from '$app/env';
	import { goto } from '$app/navigation';
	import { PLACEHOLDER_TEXT, TITLE } from '$lib/constants/form';
	import user from '$lib/shared/stores/user';
	import { isSupported, register, sign } from 'u2f-api';
	import TextInputField from '../TextInputField.svelte';

	export let fidoU2fRegistered = false;
	let label = '';
	$: registering = false;

	const registerLoginButtonText = fidoU2fRegistered
		? 'Log in with FIDO U2F'
		: 'Register with FIDO U2F';

	$: submitting = false;
	let registerData;
	$: registerData = null;

	$: checkFidoU2FSupport;
	async function checkFidoU2FSupport() {
		return browser && (await isSupported());
	}

	$: fidoU2fSupported = checkFidoU2FSupport();

	async function handleAuthenticateRegister() {
		if (fidoU2fRegistered) {
			await handleAuthenticate();
		} else {
			await handleRegister();
		}
	}

	async function authenticate(fidoU2fSignRequest) {
		try {
			const signData = await sign(fidoU2fSignRequest);

			/* add code here to send the signature to your server and get a yay or nay */

			if (authorised) {
				user.set({ ...$user, mfaAuthenticated: true });
				await goto('/dashboard');
			} else {
				console.log('Access denied!');
				await goto('/login');
			}
		} catch (error) {
			console.error(`Error in authenticate function in FidoU2f: ${error}`);
		}
	}

	async function handleAuthenticate() {
		try {
			/* add code here to request an authentication challenge from your server */

			authenticate(signRequests[0]);
		} catch (error) {
			console.error(`Error in handleAuthenticate: ${error}`);
		}
	}

	async function completeRegistration() {
		try {
			/* add code here to send the registration data to your server */

			if (registrationSuccessful) {
				await goto('/dashboard');
			}
		} catch (error) {
			console.error(`Error in completeRegistration: ${error}`);
		}
	}

	async function handleRegister() {
		if (browser && fidoU2fSupported) {
			try {
				registering = true;

				/* add code here to request fidoU2fBeginRegister from your server */

				registerData = await register([fidoU2fBeginRegister]);
			} catch (error) {
				let message;
				if (error?.metaData?.type) {
					message = error.metaData.type;
				} else {
					message = error;
				}
				console.error(`Error in handleRegister: ${message}`);
			}
		}
	}
</script>

{#if fidoU2fSupported}
	{#if registering}
		<form on:submit|preventDefault={completeRegistration}>
			<TextInputField
				value={label}
				required
				placeholder={PLACEHOLDER_TEXT.fidoU2fLabel}
				id="fido-u2f-key-label"
				title={TITLE.fidoU2fLabel}
				on:update={(event) => {
					label = event.detail;
				}}
			/>
			<button type="submit" disabled={registerData == null && label === ''}
				>Complete registration</button
			>
		</form>
	{/if}
	<button on:click={handleAuthenticateRegister} disabled={submitting}
		>{registerLoginButtonText}</button
	>
{:else}
	<div>FIDO U2F is not supported on your browser</div>
{/if}
