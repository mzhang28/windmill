<script lang="ts">
	import { ConfigService, type AutoscalingEvent } from '$lib/gen'
	import { LoaderIcon, RefreshCw } from 'lucide-svelte'
	import { Button, Skeleton } from './common'
	import { twMerge } from 'tailwind-merge'
	import TimeAgo from './TimeAgo.svelte'
	import { enterpriseLicense } from '$lib/stores'

	export let worker_group: string

	let loading = true
	let events: AutoscalingEvent[] | undefined = undefined

	$: worker_group && loadEvents()

	async function loadEvents() {
		loading = true
		try {
			events = await ConfigService.listAutoscalingEvents({ workerGroup: worker_group })
		} catch (e) {
			events = []
			console.error(e)
		} finally {
			loading = false
		}
	}
</script>

<div>
	<h6
		class={!$enterpriseLicense || (events != undefined && events.length == 0)
			? 'text-tertiary'
			: ''}
		>Autoscaling events {#if $enterpriseLicense}<span class="text-xs text-tertiary">(5 last)</span>
			<span class="inline-flex ml-6">
				<Button
					startIcon={{
						icon: loading ? LoaderIcon : RefreshCw,
						classes: twMerge(
							loading ? 'animate-spin text-blue-800' : '',
							'transition-all text-gray-500 dark:text-white'
						)
					}}
					color="light"
					size="xs2"
					btnClasses={twMerge(loading ? ' bg-blue-100 dark:bg-blue-400' : '', 'transition-all')}
					on:click={() => loadEvents()}
					iconOnly
				/>
			</span>{/if}
	</h6>
	{#if !$enterpriseLicense}
		<div class="text-xs pt-2 text-tertiary">Autoscaling is an EE feature</div>
	{:else if loading}
		<Skeleton layout={[[12], 1]} />
	{:else if events}
		{#if events.length == 0}
			<div class="text-xs pt-2 text-tertiary"
				>No events, is autoscaling set in the worker group config?</div
			>
		{:else}
			<div class="flex flex-col gap-2 text-xs text-tertiary pt-4">
				{#each events as event}
					<div class="flex flex-row gap-4">
						<div class="text-primary">{event.event_type} to {event.desired_workers}</div>
						<div class="text-secondary">{event.reason}</div>
						<div class="text-tertiary"><TimeAgo date={event.applied_at ?? ''} /></div>
					</div>
				{/each}
			</div>
		{/if}
	{/if}
</div>
