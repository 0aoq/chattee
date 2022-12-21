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

	let userBadges = [] as any;

	onMount(async () => {
		// load user
		try {
			user = await pb.collection("users").getFirstListItem(`username = "${username}"`);

			// load user badges
			userBadges = await pb.collection("user_badges").getFirstListItem(`user = "${user.id}"`);
			userBadges = userBadges.badges;
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
			<div class="mb-8">
				<h1>{username}</h1>

				<div class="flex" style="gap: var(--u-2);">
					{#each userBadges as badge}
						<span class="chip">{badge}</span>
					{/each}
				</div>
			</div>

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
		transform: scale(1.02);
		box-shadow: 0 0 8px rgba(0, 0, 0, 0.25);
	}

	.pfp:active {
		transform: scale(1.02) translateY(2px);
	}

	span.chip {
		color: hsl(0, 0%, 0%);
		background: hsl(208, 8%, 93%);
		border-radius: 360px;
		padding: 0.2rem 1rem;
		font-size: 0.8rem;
		cursor: pointer;
		user-select: none;
		transition: all 0.05s;
	}
	span.chip:hover {
		background: hsl(208, 8%, 87%);
		text-decoration: none;
		color: hsl(0, 0%, 0%);
	}
</style>
