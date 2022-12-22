<script lang="ts">
	import Footer from "$lib/components/Footer.svelte";

	import PocketBase, { Record } from "pocketbase";
	import type { PageData } from "../$types";
	import { onMount } from "svelte";

	export let data: PageData;
	// @ts-ignore
	export const { host, username } = data;

	// create pb client
	const pb = new PocketBase(`${window.location.protocol}//${host}`);

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

	// handle changing profile picture
	async function changePfp() {
		// get new image...
		if (!pb.authStore.model) return;
		if (!pb.authStore.model.id === (user.id as any)) return;

		// create input
		const input = document.createElement("input");
		input.type = "file";
		input.accept = "image/jpg, image/jpeg, image/png, image/svg+xml, image/gif";
		input.click();

		// wait for change
		input.addEventListener("change", async () => {
			if (!input.files || !pb.authStore.model) return;
			let file = input.files[0];

			// upload
			const data = new FormData();
			data.set("avatar", file);

			await pb.collection("users").update(pb.authStore.model.id, data);
			window.location.reload();
		});
	}

	// handle bio editing
	let bioEditMode = false;
	let bioEditContent: HTMLElement;

	async function saveBio(e: any) {
		if (!pb.authStore.model) return;
		if (!bioEditContent) return;

		// update user bio
		await pb.collection("users").update(pb.authStore.model.id, {
			bio: bioEditContent.innerText
		});

		user.bio = bioEditContent.innerText;

		// remove bio editor
		bioEditMode = false;
	}
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

				{#if pb.authStore.model && user.id === pb.authStore.model.id}
					<section class="mb-2">
						Actions:

						<div class="flex mt-2" style="gap: var(--u-2);">
							<button
								style="width: max-content;"
								class="primary"
								on:click={() => {
									// clear auth state and reload
									pb.authStore.clear();
									window.location.href = `/${host}/auth`;
								}}>Sign Out</button
							>
						</div>
					</section>
				{/if}

				{#if userBadges.length > 0}
					<section>
						<span class="flex align-center" style="gap: var(--u-2);">
							<svg
								xmlns="http://www.w3.org/2000/svg"
								width="18"
								height="18"
								viewBox="0 0 24 24"
								fill="none"
								stroke="var(--primary-low)"
								stroke-width="2"
								stroke-linecap="round"
								stroke-linejoin="round"
								class="feather feather-award"
								><circle cx="12" cy="8" r="7" /><polyline
									points="8.21 13.89 7 23 12 20 17 23 15.79 13.88"
								/></svg
							>

							<span>
								Badges in <i>{host}</i>
								<sup
									title="Badges help show a user's credibility in a specific server. They can only be assigned by server admins."
									style="text-decoration: underline;">&lpar;?&rpar;</sup
								>
							</span>
						</span>

						<div class="flex mt-4" style="gap: var(--u-2);">
							{#each userBadges as badge}
								<span class="chip">{badge}</span>
							{/each}
						</div>
					</section>
				{/if}
			</div>

			<div class="flex mb-4" style="gap: var(--u-4); flex-wrap: wrap;">
				<div>
					<img
						src="{window.location.protocol}//{host}/api/files/_pb_users_auth_/{user.id}/{user.avatar}?thumb=200x200"
						alt="{username}&apos;s avatar"
						title="{username}&apos;s avatar"
						class="pfp"
					/>

					{#if pb.authStore.model && user.id === pb.authStore.model.id}
						<button on:click={changePfp}>Change Picture</button>
					{/if}
				</div>

				<div>
					{#if !bioEditMode}
						<p style="max-width: var(--u-100);">{user.bio || ""}</p>
					{:else}
						<p contenteditable="true" style="max-width: var(--u-100);" bind:this={bioEditContent}>
							{user.bio || "<bio goes here>"}
						</p>

						<button class="primary mt-2" style="width: max-content;" on:click={saveBio}>Save</button
						>
					{/if}

					{#if pb.authStore.model && user.id === pb.authStore.model.id && !bioEditMode}
						<button
							class="mt-2"
							on:click={() => {
								bioEditMode = true;
							}}>Edit Bio</button
						>
					{/if}
				</div>
			</div>
		</section>

		<Footer {host} />
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
