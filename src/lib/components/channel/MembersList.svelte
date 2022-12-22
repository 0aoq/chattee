<script lang="ts">
	import type PocketBase from "pocketbase";
	import type { Record } from "pocketbase";
	import { onMount } from "svelte";

	export let pb: PocketBase; // <- we're going to create pocketbase like this so we don't have too many clients running
	export let channel: Record;
	export let host: string;

	async function kickMember(id: string) {
		// remove from members
		channel.members.splice(channel.members.indexOf(id), 1);

		// push to channel
		await pb.collection("channels").update(channel.id, {
			members: channel.members
		});

		// re-expand channel
		let c = await pb.collection("channels").getOne(channel.id, {
			expand: "members"
		});

		expanded = c.expand.members;
	}

	let expanded = undefined as any;
	onMount(async () => {
		// expand channel
		let c = await pb.collection("channels").getOne(channel.id, {
			expand: "members"
		});

		expanded = c.expand.members;
	});
</script>

<component>
	<section>
		<h1>Members In {channel.name}</h1>

		<div class="file-browser">
			{#if channel && expanded}
				{#each expanded as member}
					<div class="listing">
						<a
							class="flex justify-center align-center"
							style="gap: var(--u-4); display: flex;"
							href="/{host}/u/{member.username}?return_channel={channel.id}"
						>
							<img
								class="pfp"
								src="{window.location.protocol}//{host}/api/files/_pb_users_auth_/{member.id}/{member.avatar}?thumb=35x35"
								alt="{member.username}'s avatar"
								loading="lazy"
							/>

							{member.username}
						</a>

						<div class="flex justify-center align-center" style="gap: var(--u-2);">
							{#if pb.authStore.model && channel && channel.owner === pb.authStore.model.id}
								{#if member.id !== pb.authStore.model.id}
									<button
										on:click={() => {
											kickMember(member.id);
										}}>Remove</button
									>
								{:else}
									<i>&lpar;self&rpar;</i>
								{/if}
							{/if}
						</div>
					</div>
				{:else}
					<p>No members found!</p>
				{/each}
			{/if}
		</div>
	</section>
</component>

<style>
	button {
		width: max-content;
	}

	.pfp {
		border-radius: 360rem;
		box-shadow: 0 0 8px rgba(0, 0, 0, 0.25);
	}
</style>
