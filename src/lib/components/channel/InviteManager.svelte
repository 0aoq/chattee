<script lang="ts">
	import type PocketBase from "pocketbase";
	import type { Record } from "pocketbase";

	export let pb: PocketBase; // <- we're going to create pocketbase like this so we don't have too many clients running
	export let channel: Record;

	async function deleteInvite(id: string) {
		// remove invite from channel
		channel.invites.splice(channel.invites.indexOf(id), 1);

		// send request to update channel
		await pb.collection("channels").update(channel.id, {
			invites: channel.invites
		});

        channel = channel; // <- trigger ui update
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

					<div>
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
