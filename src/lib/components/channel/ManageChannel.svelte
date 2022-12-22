<script lang="ts">
	import type PocketBase from "pocketbase";
	import type { Record } from "pocketbase";

	export let pb: PocketBase; // <- we're going to create pocketbase like this so we don't have too many clients running
	export let channel: Record;
	export let host: string;
	export let updatePage: (v: string) => void;

	async function changeName(e: any) {
		e.preventDefault();

		// send request
		await pb.collection("channels").update(channel.id, {
			name: e.target.name.value
		});

		// refresh
		window.location.reload();
	}

	async function deleteChannel() {
		try {
			await pb.collection("channels").delete(channel.id);
			window.location.href = `/${host}/`;
		} catch {
			alert("Failed to delete channel!");
		}
	}
</script>

{#if pb.authStore.model && pb.authStore.model.id === channel.owner}
	<component>
		<section>
			<h1>Manage {channel.name}</h1>

			<div class="file-browser">
				<form class="listing" on:submit={changeName}>
					<input type="text" placeholder="Channel Name" name="name" value={channel.name || ""} />
					<button>Change Name</button>
				</form>

				<div class="listing">
					<p class="flex justify-center align-center">Delete Channel</p>
					<button on:click={deleteChannel}>Delete</button>
				</div>

				<div class="listing">
					<p class="flex justify-center align-center">Manage Members</p>
					<button
						on:click={() => {
							updatePage("members");
						}}>View</button
					>
				</div>

				<div class="listing">
					<p class="flex justify-center align-center">Manage Invites</p>
					<button
						on:click={() => {
							updatePage("invites");
						}}>View</button
					>
				</div>
			</div>
		</section>
	</component>
{/if}

<style>
	button {
		width: max-content;
	}
</style>
