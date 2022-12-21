<script lang="ts">
	import type PocketBase from "pocketbase";
	import type { Record } from "pocketbase";

	export let pb: PocketBase; // <- we're going to create pocketbase like this so we don't have too many clients running
	export let channel: Record;
	export let host: string;

	async function deleteInvite(id: string) {
		// remove invite from channel
		channel.invites.splice(channel.invites.indexOf(id), 1);

		// send request to update channel
		await pb.collection("channels").update(channel.id, {
			invites: channel.invites
		});

        channel = channel; // <- trigger ui update

		// now delete invite (for real)
		await pb.collection("invites").delete(id);
	}
</script>

<component>
	<section>
		<h1>Invites For {channel.name}</h1>

		<div class="file-browser">
			{#each channel.invites as invite}
				<div class="listing">
					<span class="flex justify-center align-center">
						{invite}
					</span>

					<div class="flex" style="gap: var(--u-2);">
						<button class="primary"
							on:click={() => {
								navigator.clipboard.writeText(`${window.location.origin}/${host}/invite/${invite}`);
								alert("Copied to clipboard!");
							}}>Copy</button
						>

						<button
							on:click={() => {
								deleteInvite(invite);
							}}>Delete</button
						>
					</div>
				</div>
			{:else}
				<p>No invites found!</p>
			{/each}
		</div>
	</section>
</component>

<style>
	button {
		width: max-content;
	}
</style>