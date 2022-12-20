<script lang="ts">
	import PocketBase, { Record } from "pocketbase";
	import type { PageData } from "../$types";
	import { onMount } from "svelte";

	export let data: PageData;
	// @ts-ignore
	export const { host, username } = data;

	// create pb client
	const pb = new PocketBase(`http://${host}`);

	let user: Record = {} as any;
	let returnChannel: string | null;

	onMount(async () => {
		// load user
		try {
			user = await pb.collection("users").getFirstListItem(`username = "${username}"`);
			user = user.items[0];
		} catch {
			// return home if we fail
			// window.location.href = `/${host}/`;
		}

		// set returnChannel
		returnChannel = new URLSearchParams(window.location.search).get("return_channel");
	});
</script>

<app>
	<nav>
		<div class="title">
			<a
				href="/{host}"
				class="grid place-center"
				aria-label="User: {username}"
				title="Click to return to channel list"
			>
				{username}
			</a>
		</div>

		<div>
			{#if returnChannel}
				<button
					class="primary"
					on:click={() => {
						window.location.href = `/${host}/c/${returnChannel}`;
					}}
				>
					Close User Page
				</button>
			{/if}
		</div>
	</nav>

	<main>
		<section>
			<h1>{username}</h1>

			<div class="flex mb-4" style="gap: var(--u-4); flex-wrap: wrap;">
				<img
					src="http://{host}/api/files/_pb_users_auth_/{user.id}/{user.avatar}?thumb=200x200"
					alt="{username}&apos;s avatar"
					title="{username}&apos;s avatar"
					class="pfp"
				/>

				<p>Description goes here later ahahaha, max 150 characters</p>
			</div>
		</section>

		<section class="grid place-center mt-4">
			<span>
				<a href="/{host}/">{host}</a>
			</span>
		</section>
	</main>
</app>

<style>
	.pfp {
		border-radius: 0.4rem;
		transition: all 0.2s;
	}

	.pfp:hover {
		transform: translate3d(0, 0, 50px);
		box-shadow: 0 0 8px rgba(0, 0, 0, 0.25);
	}

	.pfp:active {
		transform: scale(1.02) translateY(2px);
	}
</style>
