<script>
	/** @type {AudioContext | undefined} */
	let audioContext;

	/** @type {string | undefined} */
	let audioContextState;

	/** @type {number} */
	let audioContextCurrentTime;

	/** @type {number} */
	let sampleRate;

	/** @type {number} */
	let baseLatency;

	/** @type {number} */
	let outputLatency;

	/** @type {number} */
	let destNumOutputs;

	/** @type {number} */
	let destNumInputs;

	/** @type {OscillatorNode} */
	let oscillatorNode;

	/** @type {GainNode} */
	let gainNode;

	/** @type {boolean} */
	let makingNoise;

	/** @type {boolean} */
	let disabled;

	function initAudioContext() {
		if (!audioContext) {
			audioContext = new window.AudioContext();
			makingNoise = false;
			disabled = false;
			(function updateCurrentTime() {
				audioContextCurrentTime = audioContext.currentTime;
				audioContextState = audioContext.state;
				sampleRate = audioContext.sampleRate;
				baseLatency = audioContext.baseLatency;
				outputLatency = audioContext.outputLatency;
				destNumOutputs = audioContext.destination.numberOfOutputs;
				destNumInputs = audioContext.destination.numberOfInputs;
				requestAnimationFrame(updateCurrentTime);
			})();
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
			oscillatorNode.connect(audioContext.destination);
			oscillatorNode.start();
			makingNoise = true;
			disabled = true;
		} else {
			console.error('AudioContext not initialized');
		}
	}

	async function stopNoise() {
		if (audioContext && audioContextState === 'running') {
			oscillatorNode.stop();
			audioContextState = audioContext?.state;
			makingNoise = false;
			disabled = false;
			audioContextCurrentTime = audioContext.currentTime;
		}
	}
</script>

<button on:click={initAudioContext}>ON</button>
<button on:click={closeAudioContext}>OFF</button>

{#if !audioContext}
	<p>AudioContext not initialized</p>
{:else}
	<div>
		<button on:click={makeNoise} {disabled}>Make Noise</button>
		{#if makingNoise}
			<button on:click={stopNoise}>Stop Noise</button>
			<p>type: {oscillatorNode.type}</p>
			<p>frequency value: {oscillatorNode.frequency.value}</p>
		{/if}
	</div>
	<p>state: {audioContextState}</p>
	<p>current time: {audioContextCurrentTime}</p>
	<p>sample rate: {sampleRate}</p>
	<p>base latency: {baseLatency}</p>
	<p>output latency: {outputLatency}</p>
	<p>destination channel interpretation: {audioContext.destination.channelInterpretation}</p>
	<p>destination channel count mode: {audioContext.destination.channelCountMode}</p>
	<p>destination channel count: {audioContext.destination.channelCount}</p>
	<p>destination number of outputs: {destNumOutputs}</p>
	<p>destination number of inputs: {destNumInputs}</p>
{/if}
