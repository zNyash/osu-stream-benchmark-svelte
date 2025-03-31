<script lang="ts">
	import GithubIcon from '$lib/components/icons/GithubIcon.svelte';
	import { Button } from '$lib/components/ui/button';
	import { Input } from '$lib/components/ui/input';
	import { Label } from '$lib/components/ui/label';
	import { ToggleGroup, ToggleGroupItem } from '$lib/components/ui/toggle-group/';
	import { Tooltip, TooltipProvider, TooltipTrigger } from '$lib/components/ui/tooltip';
	import TooltipContent from '$lib/components/ui/tooltip/tooltip-content.svelte';
	import { getBpm } from '$lib/osuCalc';
	import { Clock, MousePointerClick } from '@lucide/svelte';
	import _ from 'lodash';

	const data = [
		{ x: 0, y: 1 },
		{ x: 1, y: 2 },
		{ x: 2, y: 3 }
	];
	// Magic Numbers
	const DEFAULT_KEY_1 = 'Z';
	const DEFAULT_KEY_2 = 'X';
	const DEFAULT_CLICKS = 100;
	const DEFAULT_SECONDS = 10;
	const MIN_CLICKS = 3;
	const MIN_SECONDS = 1;
	const MODE = {
		CLICKS: 'clicks',
		SECONDS: 'seconds'
	};

	let config = $state({
		mode: MODE.CLICKS,
		clicksValue: DEFAULT_CLICKS,
		secondsValue: DEFAULT_SECONDS
	});
	// Keybind States
	let key1 = $state(DEFAULT_KEY_1);
	let key2 = $state(DEFAULT_KEY_2);
	// Info States
	let BPM = $state(0);
	let UR = $state(0);
	// Benchmark Tracking States
	let isRunningBenchmark = $state(false);
	let tapCount = $state(0);
	let timestamps = $state<number[]>([]);
	// Timer States
	let startTime = $state<number | null>(null);
	let elapsedTime = $state(0);
	let timerInteval: ReturnType<typeof setInterval> | null = null;

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

	// Input Value Handling Functions
	function handleInputConfigMode(event: string) {
		if (event === MODE.CLICKS) {
			config.mode = MODE.CLICKS;
		} else {
			config.mode = MODE.SECONDS;
		}
	}
	function handleInputConfigValue(event: Event) {
		const target = event.target as HTMLInputElement | null;
		if (target) {
			const newValue = Number(target.value);
			if (config.mode === MODE.CLICKS) {
				config.clicksValue = Math.max(newValue, MIN_CLICKS);
			} else {
				config.secondsValue = Math.max(newValue, MIN_SECONDS);
			}
		}
	}

	// Benchmark Toggle Function
	function toggleIsRunningBenchmark() {
		if (!isRunningBenchmark) {
			startTime = null;
			elapsedTime = 0;
			tapCount = 0;
			BPM = 0;
			UR = 0;
			timestamps = [];
		}
		isRunningBenchmark = !isRunningBenchmark;
	}

	function handleBenchmarkKeyDown(event: KeyboardEvent) {
		// Keybind checks to Start and End benchmark
		if (!isRunningBenchmark) {
			if (event.code === 'Space' && !isRunningBenchmark) {
				event.preventDefault();
				toggleIsRunningBenchmark();
			}
			return;
		}
		if (event.key === 'Escape') {
			toggleIsRunningBenchmark();
			return;
		}

		// Keybind validation & timer start
		const pressedKey = event.key.toUpperCase();
		if (pressedKey === key1 || pressedKey === key2) {
			// Keypress timestamp
			const now = performance.now();

			if (startTime === null) {
				startTime = now;
				if (config.mode === MODE.SECONDS) {
					startTimerInterval();
				}
			}

			tapCount++;
			timestamps.push(now);
			elapsedTime = _.round((now - startTime) / 1000, 2);

			if (elapsedTime > 0) {
				BPM = _.round(getBpm(tapCount, now - startTime));
			}

			if (config.mode === MODE.CLICKS && tapCount >= config.clicksValue) {
				toggleIsRunningBenchmark();
			}
		}
	}

	function startTimerInterval() {
		if (timerInteval) clearInterval(timerInteval); // Verify if theres any interval on going for some reason just for safety
		const endTime = (startTime ?? performance.now()) + config.secondsValue * 1000;
		timerInteval = setInterval(() => {
			const now = performance.now();
			elapsedTime = _.round(((now - (startTime ?? now)) / 1000), 2);
			if (now >= endTime) {
				if (timerInteval) {
					clearInterval(timerInteval);
					timerInteval = null;
				}
				toggleIsRunningBenchmark();
			}
		}, 16.6666667);
	}
</script>

<!-- Keydown Event Listener -->
<svelte:window on:keydown={handleBenchmarkKeyDown} />

<!-- Page -->
<div class="mb-12 flex flex-col items-center">
	<section class="relative flex w-full max-w-[1300px] flex-row md:justify-center">
		<h1 class="mt-2 pl-3 pt-1.5 text-2xl font-medium md:text-4xl">osu! Tapping Benchmark</h1>
		<Button
			variant={'ghost'}
			class="absolute right-3 top-3 cursor-pointer"
			onclick={() => {
				window.open('https://www.youtube.com/watch?v=dQw4w9WgXcQ');
			}}
		>
			My Stats
		</Button>
	</section>

	<section class="section mt-24">
		<div class="flex flex-col">
			<Label for="inputAmout"
				>Stop at
				{#if config.mode === MODE.CLICKS}
					<span>{config.clicksValue} clicks</span>
				{:else}
					<span>{config.secondsValue} seconds</span>
				{/if}</Label
			>
			<div class="flex">
				<Input
					type="number"
					id="inputAmount"
					min={config.mode === MODE.CLICKS ? MIN_CLICKS : MIN_SECONDS}
					class="input rounded-r-none"
					value={config.mode === MODE.CLICKS ? config.clicksValue : config.secondsValue}
					oninput={(e) => handleInputConfigValue(e)}
				/>
				<ToggleGroup type="single" variant="nsh" value={config.mode} class="gap-0">
					<ToggleGroupItem
						value={MODE.CLICKS}
						class={`px-2 first:rounded-none ${config.mode === MODE.CLICKS ? 'bg-zinc-700 hover:bg-zinc-700' : ''}`}
						onclick={() => {
							handleInputConfigMode(MODE.CLICKS);
						}}><MousePointerClick /></ToggleGroupItem
					>
					<ToggleGroupItem
						value={MODE.SECONDS}
						class={`px-2 last:rounded-l-none ${config.mode === MODE.SECONDS ? 'bg-zinc-700 hover:bg-zinc-700' : ''}`}
						onclick={() => {
							handleInputConfigMode(MODE.SECONDS);
						}}><Clock /></ToggleGroupItem
					>
				</ToggleGroup>
			</div>
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
			<p>{tapCount} Taps | {elapsedTime} Seconds</p>
			<TooltipProvider>
				<Tooltip>
					<TooltipTrigger>
						<Button
							class="mt-4 cursor-pointer bg-sky-600 text-foreground hover:bg-sky-600/75"
							onclick={toggleIsRunningBenchmark}
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

	<Button
		class="fixed bottom-2 right-2 cursor-pointer rounded-md bg-zinc-800 text-foreground hover:bg-zinc-800/75"
		onclick={() => {
			window.open('https://github.com/zNyash/osu-stream-benchmark');
		}}
		target="_blank"><GithubIcon /> Repository</Button
	>
</div>
