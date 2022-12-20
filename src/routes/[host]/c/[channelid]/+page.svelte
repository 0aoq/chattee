<script lang="ts">
	// @ts-nocheck

	import PocketBase, { Record } from "pocketbase";
	import type { PageData } from "./$types";
	import { onMount } from "svelte";

	export let data: PageData;
	// @ts-ignore
	export const { host, channelid } = data;

	// create pb client
	const pb = new PocketBase(`http://${host}`);

	let messages: Array<Record> = [];

	let channel: Record = {} as any;
	let channelOwner = {} as any;

	onMount(async () => {
		// load messages
		const msgs = await pb.collection("messages").getList(1, 50, {
			sort: "created",
			expand: "sender"
		});

		messages = msgs.items;

		// load channel
		channel = await pb.collection("channels").getOne(channelid, {
			expand: "owner"
		});

		channelOwner = channel.expand.owner;

		// subscribe to changes to the "messages" channel
		await pb.collection("messages").subscribe("*", async (data) => {
			if (data.record.channel !== channelid) return; // <- message must be for this channel!

			// add to messages
			const msgFull = await pb.collection("messages").getOne(data.record.id, {
				expand: "sender"
			});

			messages = [...messages, msgFull]; // not using .push so svelte can update it
		});
	});

	async function deleteChannel() {
		try {
			await pb.collection("channels").delete(channelid);
			window.location.href = `/${host}/`;
		} catch {
			alert("Failed to delete channel!");
		}
	}

	async function sendMessage(e: any) {
		e.preventDefault();
		if (!pb.authStore.model) return; // <- we need a user!
		if (e.target.message.value.trim() === "") return; // <- message can't be blank!

		try {
			// try to send message
			await pb.collection("messages").create({
				channel: channelid,
				content: e.target.message.value.trim(),
				sender: pb.authStore.model.id
			});

			e.target.message.value = "";
		} catch {
			alert("Failed to send message!");
		}
	}
</script>

<app>
	<nav>
		<div class="title">
			<a
				href="/{host}"
				class="grid place-center"
				aria-label="Channel Name: {channel.name || ''}"
				title="Click to return to channel list"
			>
				{channel.name || "!"}
			</a>
		</div>

		<div>
			{#if pb.authStore.model && channel && channel.expand && pb.authStore.model.id === channelOwner.id}
				<!-- does have account, is a member of this channel, and is the owner of this channel -->
				<button
					on:click={() => {
						deleteChannel();
					}}>Delete Channel</button
				>
			{:else if pb.authStore.model && channel && channel.members && channel.members.includes(pb.authStore.model.id)}
				<!-- does have account, is a member of this channel, but isn't the owner -->
				<button>Leave Channel</button>
			{:else if pb.authStore.model && channel}
				<!-- does have account, but isn't a member of this channel -->
				<button
					class="primary"
					on:click={() => {
						window.location.href = `/${host}/`;
					}}>View Channels</button
				>
			{:else}
				<!-- does not have account -->
				<button
					class="primary"
					on:click={() => {
						window.location.href = `/${host}/auth`;
					}}>Manage Account</button
				>
			{/if}
		</div>
	</nav>

	<main>
		<section class="flex justify-center mb-4" style="gap: var(--u-2); flex-wrap: wrap;">
			{#if pb.authStore.model}
				<div class="file-browser" style="width: 100%;">
					{#each messages as msg}
						<div class="listing">
							<span class="flex">
								<b
									><a href="/{host}/u/{msg.expand.sender.username}?return_channel={channelid}"
										>{msg.expand.sender.username}</a
									></b
								>:
								{msg.content}</span
							>

							<span
								title="Created: {new Date(msg.created).toLocaleString()}, Updated: {new Date(
									msg.updated
								).toLocaleString()}"
							>
								{new Date(msg.created).toLocaleDateString()}
							</span>
						</div>
					{:else}
						<p>No message results!</p>
					{/each}
				</div>
			{:else}
				<p>Please create an account or sign in!</p>
			{/if}
		</section>

		<section>
			<form
				class="flex justify-center"
				style="flex-wrap: wrap; gap: var(--u-2);"
				on:submit={sendMessage}
			>
				<input
					type="text"
					name="message"
					required
					style="width: calc(100% - var(--u-20) - var(--u-2) * 10);"
					autocomplete="off"
				/>

				<button style="width: var(--u-20);">Send</button>
			</form>
		</section>

		<section class="grid place-center mt-4">
			<span>
				<a href="/{host}/">{host}</a>
			</span>
		</section>
	</main>
</app>
