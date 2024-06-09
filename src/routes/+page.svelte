<script lang="ts">
	import { onMount } from 'svelte';

	let code: string = $state(`puts "Hello, world!"\nputs 1.upto(5).map { |n| n * 2 }.join(", ")`);
	let loaded: boolean = $state(false);
	let logs: string[] = $state([]);
	let output = $derived(logs.join(''));

	function run(e: Event) {
		e.preventDefault();

		logs = [];
		try {
			(window as any).rubyVM.eval(code);
		} catch (e) {
			logs.push((e as Error).message);
		}
	}

	function handleCmdEnter(e: KeyboardEvent) {
		if (e.key === 'Enter' && e.metaKey) {
			run(e);
		}
	}

	onMount(async () => {
		const originalConsoleLog = console.log;

		window.console.log = function (message) {
			logs.push(message);
			originalConsoleLog.apply(console, [...arguments]);
		};

		const int = setInterval(() => {
			if ((window as any).rubyVM) {
				loaded = true;
				logs = [];
				clearInterval(int);
			}
		}, 100);
	});
</script>

<h1>Ruby on Svelte!</h1>

{#if loaded}
	<form onsubmit={run}>
		<div>
			<button type="submit">Run</button>
		</div>
		<textarea
			rows="20"
			cols="50"
			bind:value={code}
			onkeydown={handleCmdEnter}
			placeholder="Type Ruby code here"
		></textarea>
		<textarea rows="20" cols="50" value={output} readonly placeholder="Output"></textarea>
	</form>
{:else}
	<p>Loading Ruby.wasm ...</p>
{/if}

{@html `<script src="https://cdn.jsdelivr.net/npm/@ruby/3.3-wasm-wasi@2.6.1/dist/browser.script.iife.js"></script>`}

{@html `<script type="text/ruby">
    require "js"

    puts RUBY_VERSION # (Printed to the Web browser console)
  # JS.global[:document].write "Hello, world!"
  </script>`}
