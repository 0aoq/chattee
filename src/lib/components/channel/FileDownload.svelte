<script lang="ts">
	export let file: string;
	export let url: string;
</script>

<component>
	<a href={url} download>
		<section
			class="flex justify-center align-center"
			style="width: var(--u-100); gap: var(--u-8);"
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				width="24"
				height="24"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
				class="feather feather-download"
				><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" /><polyline
					points="7 10 12 15 17 10"
				/><line x1="12" y1="15" x2="12" y2="3" /></svg
			>

			{file}
		</section>
	</a>
</component>

<style>
	section {
		box-shadow: 0 0 0px rgba(0, 0, 0, 0.25);
		transition: all 0.2s;
		border-radius: 0.4rem;
	}

	section:hover {
		box-shadow: 0 0 8px rgba(0, 0, 0, 0.25);
		border-color: transparent;
		transform: scale(1.01);
	}
</style>