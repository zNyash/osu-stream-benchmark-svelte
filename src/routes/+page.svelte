<script lang="ts">
	import { Button } from '$lib/components/ui/button';
	import { Input } from '$lib/components/ui/input';
	import { ToggleGroup, ToggleGroupItem } from '$lib/components/ui/toggle-group/';
	import { Tooltip, TooltipProvider, TooltipTrigger } from '$lib/components/ui/tooltip';
	import TooltipContent from '$lib/components/ui/tooltip/tooltip-content.svelte';
	import { Clock, MousePointerClick } from '@lucide/svelte';

	// Magic Numbers
	const DEFAULT_KEY_1 = 'Z';
	const DEFAULT_KEY_2 = 'X';

	// Keybind States
	let key1 = $state(DEFAULT_KEY_1);
	let key2 = $state(DEFAULT_KEY_2);
	// Info
	let BPM = $state(0);
	let UR = $state(0);
	// Benchmark tracking
	let isRunningBenchmark = $state(false);

	// Keybind Functions
	function handleKeybind(event: Event, key: string) {
		const target = event.currentTarget as HTMLInputElement | null;
		if (target) {
			key = (target.value.at(-1) || '').toUpperCase();
			target.value = key;
		}
	}
	function handleKeybindFocus(event: Event) {
		const target = event.target as HTMLInputElement;
		target.select();
	}
</script>

<div class="mb-12 flex flex-col items-center">
	<section class="relative flex w-full max-w-[1300px] flex-row md:justify-center">
		<h1 class="mt-2 pl-3 pt-1.5 text-2xl font-medium md:text-4xl">osu! Tapping Benchmark</h1>
		<Button
			variant={'ghost'}
			class="absolute right-3 top-3 cursor-pointer"
			href="https://www.youtube.com/watch?v=dQw4w9WgXcQ"
			target="_blank"
		>
			My Stats
		</Button>
	</section>

	<section class="section mt-24">
		<div class="flex">
			<Input type="number" class="input rounded-r-none" />
			<ToggleGroup type="single" variant="nsh" value="clicks" class="gap-0">
				<ToggleGroupItem value="clicks" class="px-2 first:rounded-none"
					><MousePointerClick /></ToggleGroupItem
				>
				<ToggleGroupItem value="seconds" class="px-2 last:rounded-l-none"><Clock /></ToggleGroupItem
				>
			</ToggleGroup>
		</div>
	</section>

	<section class="section mt-2">
		<div>
			<p>Choose your keybinds:</p>
			<div class="flex flex-row justify-center gap-1">
				<input
					type="text"
					maxlength="2"
					class="input size-9 rounded-md border text-center align-top uppercase leading-none caret-transparent"
					id="key1"
					value={key1}
					oninput={(e) => {
						handleKeybind(e, key1);
					}}
					onfocus={handleKeybindFocus}
				/>
				<input
					type="text"
					maxlength="2"
					class="input size-9 rounded-md border text-center align-top uppercase leading-none caret-transparent"
					id="key2"
					value={key2}
					oninput={(e) => {
						handleKeybind(e, key2);
					}}
					onfocus={handleKeybindFocus}
				/>
			</div>
		</div>
	</section>

	<section class="section mt-12">
		<div class="flex flex-col items-center">
			<p class="text-6xl font-bold">{BPM} BPM</p>
			<TooltipProvider>
				<Tooltip>
					<TooltipTrigger>
						<Button class="text-foreground mt-4 cursor-pointer bg-sky-600 hover:bg-sky-600/75"
							>{isRunningBenchmark ? 'Stop Benchmark' : 'Start Benchmark'}</Button
						>
					</TooltipTrigger>
					<TooltipContent>
						<p>{isRunningBenchmark ? 'Press ESC to cancel' : 'Press SPACE to start'}</p>
					</TooltipContent>
				</Tooltip>
			</TooltipProvider>
		</div>
	</section>
</div>
