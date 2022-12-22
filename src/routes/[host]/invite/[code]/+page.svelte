<script lang="ts">
	import PocketBase from "pocketbase";
	import type { PageData } from "./$types";
	import { onMount } from "svelte";

	export let data: PageData;
	// @ts-ignore
	export const { host, code } = data;

	// create pb client
	let pb = new PocketBase(`http://${host}`);

	onMount(async () => {
		if (window.location.protocol === "https:") pb = new PocketBase(`https://${host}`);

		if (!pb.authStore.model) return (window.location.href = `/${host}/auth`); // <- we need a user!

		try {
			// get invite
			const invite = await pb.collection("invites").getOne(code);

			// set invite.accepted to include our id
			await pb.collection("invites").update(invite.id, {
				accepted: [...(invite.accepted || []), pb.authStore.model.id]
			});

			// get channel connected to the invite
			const channel = await pb.collection("channels").getOne(invite.channel);

			// add ourselves to the channel (we can do that now that we accepted the invite!)
			await pb.collection("channels").update(channel.id, {
				members: [...channel.members, pb.authStore.model.id]
			});

			// navigate to this channel
			window.location.href = `/${host}/c/${channel.id}`;
		} catch {
			window.location.href = `/${host}/`;
		}
	});
</script>
