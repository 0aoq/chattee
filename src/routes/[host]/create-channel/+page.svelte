<script lang="ts">
	import Footer from "$lib/components/Footer.svelte";

	import PocketBase from "pocketbase";
	import type { PageData } from "./$types";
	import { onMount } from "svelte";

	export let data: PageData;
	// @ts-ignore
	export const { host } = data;

	// create pb client
	const pb = new PocketBase(`${window.location.protocol}//${host}`);

	// handle form submit
	let errorMessage = "";

	// create function
	async function create(e: any) {
		e.preventDefault();
		if (!pb.authStore.model) return; // <- no account!

		try {
			// send create request
			const r = await pb.collection("channels").create({
				name: e.target.name.value,
				owner: pb.authStore.model.id, // <- we are the owner!
				members: [pb.authStore.model.id] // <- we are also the first member!
			});

			window.location.href = `/${host}/c/${r.id}`;
		} catch (err: any) {
			errorMessage = err.toString();
		}
	}
</script>

<app>
	<main>
		<section>
			<h1>Create Channel</h1>

			{#if pb.authStore.model}
				<div class="grid place-center">
					<form
						class="flex mt-12 mb-8"
						style="flex-direction: column; width: var(--u-100);"
						on:submit|preventDefault={create}
					>
						<label for="name" class="mb-2"><b>Channel Name&ast;</b></label>
						<input type="text" name="name" required placeholder="Channel Name" />

						<button class="primary mt-4">Create</button>

						<p class="mt-4" style="color: red;">{errorMessage}</p>
					</form>
				</div>
			{:else}
				<p>Please create an account or sign in to create channels on this instance!</p>
			{/if}
		</section>

		<Footer {host} />
	</main>
</app>
