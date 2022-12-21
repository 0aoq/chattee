<script lang="ts">
	import Footer from "$lib/components/Footer.svelte";

	import PocketBase, { Record } from "pocketbase";
	import type { PageData } from "./$types";
	import { onMount } from "svelte";

	export let data: PageData;
	// @ts-ignore
	export const { host } = data;

	// create pocketbase client
	const pb = new PocketBase(`http://${host}`);
	let myChannels: Array<Record> = [];

	onMount(async () => {
		if (!pb.authStore.model) return;

		// fetch channels
		// maximum of 50, page 1, sort by -created
		const channels = await pb.collection("channels").getList(1, 50, {
			sort: `-created`
		});

		myChannels = channels.items;
	});
</script>

<app>
	<main>
		<section class="mb-4 flex justify-center" style="gap: var(--u-2);">
			<a
				href={pb.authStore.model === null
					? `/${host}/auth`
					: `/${host}/u/${pb.authStore.model.username}`}
				class="chip">Account</a
			>
			<a href="/{host}/create-channel" class="chip">Create Channel</a>
		</section>

		<section class="flex justify-center" style="gap: var(--u-2); flex-wrap: wrap;">
			{#if pb.authStore.model}
				<div class="file-browser" style="width: 100%;">
					{#each myChannels as channel}
						<div class="listing">
							<a
								href="/{host}/c/{channel.id}"
								class="flex justify-center align-center"
								style="display: flex; gap: var(--u-2);"
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									width="18"
									height="18"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="2"
									stroke-linecap="round"
									stroke-linejoin="round"
									class="feather feather-message-square"
									><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z" /></svg
								>

								{channel.name}
							</a>
						</div>
					{:else}
						<p>👻 No channels found...</p>
					{/each}
				</div>
			{:else}
				<p>Please create an account or sign in!</p>
			{/if}
		</section>

		<Footer {host} />
	</main>
</app>
