<script lang="ts">
	// @ts-nocheck
	import InviteManager from "$lib/components/channel/InviteManager.svelte";
	import MembersList from "$lib/components/channel/MembersList.svelte";
	import Message from "$lib/components/channel/Message.svelte";
	import Footer from "$lib/components/Footer.svelte";

	import PocketBase, { Record } from "pocketbase";
	import type { PageData } from "./$types";
	import { onMount } from "svelte";

	export let data: PageData;
	// @ts-ignore
	export const { host, channelid } = data;

	// create pb client
	let pb = new PocketBase(`http://${host}`);

	let messages: Array<Record> = [];

	let channel: Record = {} as any;
	let channelOwner = {} as any;

	onMount(async () => {
		if (window.location.protocol === "https:") pb = new PocketBase(`https://${host}`);

		// load messages
		const msgs = await pb.collection("messages").getList(1, 50, {
			sort: "created",
			expand: "sender"
		});

		messages = msgs.items;

		setTimeout(() => {
			// scroll to the bottom so the last message is in view
			window.scrollTo(0, document.body.scrollHeight);
		}, 500);

		// load channel
		channel = await pb.collection("channels").getOne(channelid, {
			expand: "owner"
		});

		channelOwner = channel.expand.owner;

		// subscribe to changes to the "messages" channel
		await pb.collection("messages").subscribe("*", async (data) => {
			if (data.record.channel !== channelid) return; // <- message must be for this channel!

			// add to messages
			try {
				const msgFull = await pb.collection("messages").getOne(data.record.id, {
					expand: "sender"
				});

				messages = [...messages, msgFull]; // not using .push so svelte can update it
			} catch {
				// message got deleted, reset list
				const msgs = await pb.collection("messages").getList(1, 50, {
					sort: "created",
					expand: "sender"
				});

				messages = msgs.items;
			}
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

	let attachedFile: File | null = null;
	let messageSendError = "";
	async function sendMessage(e: any) {
		e.preventDefault();
		if (!pb.authStore.model) return; // <- we need a user!
		if (e.target.message.value.trim() === "") return; // <- message can't be blank!
		if (!channel.members.includes(pb.authStore.model.id)) return; // <- must be a member of this channel!

		try {
			// collect data
			const data = new FormData();
			data.set("channel", channelid);
			data.set("content", e.target.message.value.trim());
			data.set("sender", pb.authStore.model.id);

			if (attachedFile) {
				data.append("attachment", attachedFile);

				const n = attachedFile.name.toLowerCase();
				data.set(
					"attachmentType",
					n.endsWith(".mp4") || n.endsWith(".webm") || n.endsWith(".mov")
						? "video"
						: n.endsWith(".png") || n.endsWith(".jpg") || n.endsWith(".jpeg") || n.endsWith(".svg")
						? "image"
						: "file"
				);
			}

			// try to send message
			await pb.collection("messages").create(data);

			e.target.message.value = "";
			attachedFile = null;
		} catch (err: any) {
			messageSendError = "Message must be less than 1000 character(s).";

			setTimeout(() => {
				messageSendError = "";
			}, 2000);
		}
	}

	async function createInvite() {
		try {
			const invite = await pb.collection("invites").create({
				channel: channelid,
				accepted: []
			});

			await pb.collection("channels").update(channelid, {
				// add our invite id to channel.invites
				invites: [...(channel.invites || []), invite.id]
			});

			channel = await pb.collection("channels").getOne(channelid); // <- trigger InviteManager UI update

			alert(`${window.location.origin}/${host}/invite/${invite.id}`);
		} catch {
			alert("Failed to create invite!");
		}
	}

	async function leaveChannel() {
		try {
			// remove from array
			channel.members.splice(channel.members.indexOf(pb.authStore.model.id, 1));

			// push to server
			await pb.collection("channels").update(channelid, {
				members: channel.members
			});
		} catch {
			alert("Failed to leave channel!");
		}
	}

	async function attachFile() {
		// create file input
		const input = document.createElement("input");
		input.setAttribute("type", "file");

		// open picker dialog
		input.click();

		// listen for changes to the file status, load file once added
		input.addEventListener("change", () => {
			if (!input.files[0]) return;
			attachedFile = input.files[0];
		});
	}

	// handle page manager
	let page = "default";
</script>

<svelte:head>
	<title>Viewing Channel - Chattee!</title>
</svelte:head>

<app>
	<nav>
		<div class="title">
			<a
				href="/{host}"
				class="grid place-center"
				aria-label="Channel Name: {channel.name || ''}"
				title="Click to return to channel list"
			>
				{channel.name || "Loading!"}
			</a>
		</div>

		<div>
			{#if pb.authStore.model && channel && channel.expand && pb.authStore.model.id === channelOwner.id}
				<!-- does have account, is a member of this channel, and is the owner of this channel -->
				<button on:click={deleteChannel}>Delete Channel</button>
				<button
					on:click={() => {
						page = "invites";
					}}>Manage Invites</button
				>
				<button
					on:click={() => {
						page = "members";
					}}>Manage Members</button
				>
			{:else if pb.authStore.model && channel && channel.members && channel.members.includes(pb.authStore.model.id)}
				<!-- does have account, is a member of this channel, but isn't the owner -->
				<button on:click={leaveChannel}>Leave Channel</button>

				<button
					on:click={() => {
						page = "members";
					}}>View Members</button
				>
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

			{#if pb.authStore.model && channel && channel.expand && channel.members && channel.members.includes(pb.authStore.model.id)}
				<button class="primary" on:click={createInvite}>Create Invite</button>
			{/if}
		</div>
	</nav>

	{#if pb.authStore.model && (channel.members || []).includes(pb.authStore.model.id)}
		<main>
			{#if page === "default"}
				<section class="flex justify-center mb-4" style="gap: var(--u-2); flex-wrap: wrap;">
					{#if pb.authStore.model}
						<div class="file-browser" style="width: 100%;">
							{#each messages as msg}
								<Message {msg} {host} {pb} {channel} {channelOwner} />
							{:else}
								<p>No message results!</p>
							{/each}
						</div>
					{:else}
						<p>Please create an account or sign in!</p>
					{/if}
				</section>

				<section class="flex justify-center" style="flex-wrap: wrap; gap: var(--u-2);">
					<button style="width: var(--u-20);" on:click={attachFile}>
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
							class="feather feather-upload"
							><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" /><polyline
								points="17 8 12 3 7 8"
							/><line x1="12" y1="3" x2="12" y2="15" /></svg
						>
					</button>

					<form
						class="flex justify-space-between"
						style="flex-wrap: wrap; width: calc(var(--u-100) * 1.8);"
						on:submit={sendMessage}
					>
						<input
							type="text"
							name="message"
							required
							style="width: calc(100% - var(--u-24) - 1.4rem);"
							autocomplete="off"
						/>

						<button style="width: var(--u-24); margin-left: -2rem;" class="primary">
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
								class="feather feather-send"
								><line x1="22" y1="2" x2="11" y2="13" /><polygon
									points="22 2 15 22 11 13 2 9 22 2"
								/></svg
							>
						</button>
					</form>

					<p style="color: red;">{messageSendError}</p>
				</section>
			{:else if page === "invites"}
				<InviteManager {pb} {channel} {host} />

				<section class="mt-4 flex justify-center">
					<button
						class="primary"
						style="width: max-content;"
						on:click={() => {
							page = "default";
						}}>Return to Messages</button
					>
				</section>
			{:else if page === "members"}
				<MembersList {pb} {channel} {host} />

				<section class="mt-4 flex justify-center">
					<button
						class="primary"
						style="width: max-content;"
						on:click={() => {
							page = "default";
						}}>Return to Messages</button
					>
				</section>
			{/if}

			<Footer {host} />
		</main>
	{/if}
</app>
