<script lang="ts">
	export let host: string;

	let taglineThing = [
		"<Chattee> is an open-source UI and PocketBase schema for developing a realtime chat app...",
		"",
		"Chattee is licensed under the MIT license. The code can be found by clicking this link!"
	].join("\n");
</script>

<section class="grid place-center mt-4">
	<span>
		<a href="/{host}/">{host}</a>
		-
		<i
			><a
				href="https://github.com/0aoq/chattee"
				target="_blank"
				rel="noreferrer"
				title={taglineThing}>☕ chattee</a
			></i
		>
	</span>
</section>
