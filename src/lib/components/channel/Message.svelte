<script lang="ts">
	import type PocketBase from "pocketbase";
	import FileDownload from "./FileDownload.svelte";

	export let msg: any;
	export let host: string;

	export let pb: PocketBase;
	export let channel: any;
	export let channelOwner: any;

	// must match this rule: @request.auth.id = sender.id  || @request.auth.id = channel.owner.id
	const canDeleteThisMessage =
		(pb.authStore.model &&
			channel &&
			channel.expand &&
			pb.authStore.model.id === channelOwner.id) ||
		(pb.authStore.model && channel && pb.authStore.model.id === msg.sender);

	async function deleteMessage() {
		// delete current message
		await pb.collection("messages").delete(msg.id);
	}

	function parseMessage(msg: string): string {
		if (!pb.authStore.model) return "failed";

		// escape < and >
		msg = msg.replaceAll(/</gm, "&lt;");
		msg = msg.replaceAll(/>/gm, "&gt;");

		// decode uri
		// msg = decodeURIComponent(msg);

		// add mention thing
		if (msg.includes(`(u/${pb.authStore.model.username})`))
			msg = `<span style="outline: solid 1px var(--primary);">${msg}</span>`;

		// user tags
		msg = msg.replaceAll(
			/\(u\/(?<USER>.*)\)/gm,
			`<a href="/${host}/u/$1?return_channel=${channel.id}" style="color: var(--primary); display: inline;">@$1</a>`
		);

		// channel tags
		msg = msg.replaceAll(
			/\(c\/(?<CHANNEL>.*)\)/gm,
			`<a href="javascript:window.location.href='/${host}/c/$1'" style="color: var(--primary); display: inline;">#$1</a>`
		);

		// MARKDOWN
		msg = msg.replaceAll(/\*\*(.*)\*\*/gm, "<b>$1</b>"); // bold
		msg = msg.replaceAll(/\*(.*)\*/gm, "<i>$1</i>"); // italic
		msg = msg.replaceAll(/\_(.*)\_/gm, "<i>$1</i>"); // italic

		// return
		return msg;
	}
</script>

<div class="listing message" id={msg.id}>
	<span style="max-width: 100%;">
		<b style="display: inline-block;"
			><a href="/{host}/u/{msg.expand.sender.username}?return_channel={channel.id}"
				>{msg.expand.sender.username + " "}</a
			></b
		>: <span class="message.content" data-raw-content={msg.content}>{@html parseMessage(msg.content)}</span>

		{#if msg.attachment}
			<div>
				{#if msg.attachmentType === "video"}
					<video
						src="{window.location.protocol}//{host}/api/files/messages/{msg.id}/{msg.attachment}"
						class="attachment"
						controls
						loading="lazy"
					>
						<track kind="captions" />
					</video>
				{:else if msg.attachmentType === "image"}
					<img
						src="{window.location.protocol}//{host}/api/files/messages/{msg.id}/{msg.attachment}"
						alt={msg.attachment}
						class="attachment"
						loading="lazy"
					/>
				{:else}
					<FileDownload
						file={msg.attachment}
						url="{window.location.protocol}//{host}/api/files/messages/{msg.id}/{msg.attachment}"
					/>
				{/if}
			</div>
		{/if}
	</span>

	<span>
		<span
			title="Created: {new Date(msg.created).toLocaleString()}, Updated: {new Date(
				msg.updated
			).toLocaleString()}"
		>
			{new Date(msg.created).toLocaleDateString()}
		</span>

		{#if canDeleteThisMessage}
			<span
				class="message.action.delete"
				on:click={deleteMessage}
				on:keydown={() => {
					return;
				}}
			>
				Delete
			</span>
		{/if}
	</span>
</div>

<style>
	.attachment {
		max-width: 100%;
		max-height: var(--u-100);
	}

	@media (prefers-color-scheme: dark) {
		video {
			filter: invert(1) hue-rotate(180deg); /* undo filter */
		}
	}

	.message\.action\.delete {
		color: rgb(255, 87, 87);
	}

	.message\.action\.delete:hover {
		text-decoration: underline;
		cursor: pointer;
	}
</style>
