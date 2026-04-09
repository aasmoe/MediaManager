<script lang="ts">
	import { Button } from '$lib/components/ui/button';
	import { Input } from '$lib/components/ui/input';
	import { Label } from '$lib/components/ui/label';
	import * as Dialog from '$lib/components/ui/dialog';
	import FilePathSuffixSelector from '$lib/components/download-dialogs/file-path-suffix-selector.svelte';
	import type { components } from '$lib/api/api';
	import { toast } from 'svelte-sonner';
	import { convertTorrentSeasonRangeToIntegerRange } from '$lib/utils.ts';

	let {
		filePathSuffix = $bindable(),
		media,
		seasons,
		callback
	}: {
		filePathSuffix: string;
		media: components['schemas']['Movie'] | components['schemas']['Show'];
		/** Parsed seasons from the torrent title. Pass for TV show torrents, omit for movies. */
		seasons?: number[];
		callback: (seasonOverride?: number) => void;
	} = $props();
	let dialogOpen = $state(false);
	let seasonOverride: number | undefined = $state(undefined);

	function onDownload() {
		if (seasons !== undefined && seasons.length === 0) {
			const isValid =
				seasonOverride !== undefined && Number.isFinite(seasonOverride) && seasonOverride > 0;
			if (!isValid) {
				toast.error(
					'Could not detect a season number from the torrent title. Please enter it manually.'
				);
				return;
			}
		}
		const effectiveOverride =
			seasonOverride !== undefined && Number.isFinite(seasonOverride) && seasonOverride > 0
				? seasonOverride
				: undefined;
		callback(effectiveOverride);
		dialogOpen = false;
	}
</script>

<Dialog.Root bind:open={dialogOpen}>
	<Dialog.Trigger>
		<Button class="w-full" onclick={() => (dialogOpen = true)}>Download</Button>
	</Dialog.Trigger>
	<Dialog.Content class="w-full max-w-[600px] rounded-lg p-6 shadow-lg">
		<Dialog.Header>
			<Dialog.Title class="mb-1 text-xl font-semibold">Set File Path Suffix</Dialog.Title>
			<Dialog.Description class="mb-4 text-sm">
				Set the filepath suffix for downloaded files of the torrent.
			</Dialog.Description>
		</Dialog.Header>
		{#if seasons !== undefined}
			<div class="mb-4 grid w-full items-center gap-1.5">
				<Label for="season-override">Season Number</Label>
				{#if seasons.length > 0}
					<p class="text-sm text-muted-foreground">
						Detected season(s): {convertTorrentSeasonRangeToIntegerRange(seasons)}. You can override
						this if the detection was incorrect.
					</p>
				{:else}
					<p class="text-sm text-destructive">
						Could not detect a season number from the torrent title. Please enter it manually.
					</p>
				{/if}
				<Input
					type="number"
					id="season-override"
					class="max-w-[200px]"
					bind:value={seasonOverride}
					placeholder={seasons.length > 0 ? String(seasons[0]) : 'e.g. 1'}
					min="1"
				/>
			</div>
		{/if}
		<FilePathSuffixSelector bind:filePathSuffix {media} />
		<div class="mt-8 flex justify-between gap-2">
			<Button onclick={() => (dialogOpen = false)} variant="secondary">Cancel</Button>
			<Button onclick={() => onDownload()}>Download Torrent</Button>
		</div>
	</Dialog.Content>
</Dialog.Root>
