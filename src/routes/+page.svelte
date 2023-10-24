<script>
	/** @type {AudioContext | undefined} */
	let audioContext;

	/** @type {string | undefined} */
	let audioContextState;

	/** @type {number} */
	let audioContextCurrentTime;

	/** @type {OscillatorNode} */
	let oscillatorNode;

	/** @type {GainNode} */
	let gainNode;

	/** @type {boolean} */
	let makingNoise = false;

	function initAudioContext() {
		if (!audioContext) {
			audioContext = new window.AudioContext();
			audioContextState = audioContext.state;
			audioContextCurrentTime = audioContext.currentTime;
		}
	}

	async function closeAudioContext() {
		if (audioContext) {
			await audioContext.close();
			audioContext = undefined;
		}
	}

	async function makeNoise() {
		if (audioContext) {
			oscillatorNode = audioContext.createOscillator();
			gainNode = audioContext.createGain();

			oscillatorNode.connect(gainNode);
			gainNode.connect(audioContext.destination);
			oscillatorNode.start();
			audioContextState = audioContext.state;
			makingNoise = true;
			audioContextCurrentTime = audioContext.currentTime;
		} else {
			console.error('AudioContext not initialized');
		}
	}

	async function stopNoise() {
		if (audioContext && audioContextState === 'running') {
			oscillatorNode.stop();
			audioContextState = audioContext?.state;
			makingNoise = false;
			audioContextCurrentTime = audioContext.currentTime;
		}
	}
</script>

<button on:click={initAudioContext}>ON</button>
<button on:click={closeAudioContext}>OFF</button>

{#if !audioContext}
	<!-- content here -->
	<p>AudioContext not initialized</p>
{:else}
	<!-- else content here -->
	<p>AudioContext initialized in {audioContextState} state</p>
	<p>listener: {audioContext.listener}</p>
	<p>current time: {audioContextCurrentTime}</p>
	<p>sample rate: {audioContext.sampleRate}</p>
	<p>base latency: {audioContext.baseLatency}</p>
	<p>output latency: {audioContext.outputLatency}</p>
	<p>on state change: {audioContext.onstatechange}</p>
	<p>audio worklet: {audioContext.audioWorklet}</p>
	<p>destination: {audioContext.destination}</p>
	<p>output timestamp: {audioContext.getOutputTimestamp}</p>
	<button on:click={makeNoise}>Make Noise</button>
	<button on:click={stopNoise}>Stop Noise</button>
{/if}

{#if makingNoise}
	<p>some noise!</p>
{/if}
