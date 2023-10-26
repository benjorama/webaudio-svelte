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
			(function updateCurrentTime() {
				audioContextCurrentTime = audioContext.currentTime;
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
			gainNode.connect(audioContext.destination);
			oscillatorNode.start();
			audioContextState = audioContext.state;
			makingNoise = true;
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
	<p>AudioContext initialized</p>
	<p>state: {audioContext.state}</p>
	<p>listener: {audioContext.listener}</p>
	<p>listener upX: {audioContext.listener.upX}</p>
	<p>listener upY: {audioContext.listener.upY}</p>
	<p>listener upZ: {audioContext.listener.upZ}</p>
	<p>listener forwardX: {audioContext.listener.forwardX}</p>
	<p>listener forwardY: {audioContext.listener.forwardY}</p>
	<p>listener forwardZ: {audioContext.listener.forwardZ}</p>
	<p>listener positionX: {audioContext.listener.positionX}</p>
	<p>listener positionY: {audioContext.listener.positionY}</p>
	<p>listener positionZ: {audioContext.listener.positionZ}</p>
	<p>current time: {audioContextCurrentTime}</p>
	<p>sample rate: {audioContext.sampleRate}</p>
	<p>base latency: {audioContext.baseLatency}</p>
	<p>output latency: {audioContext.outputLatency}</p>
	<p>audio worklet: {audioContext.audioWorklet}</p>
	<p>destination: {audioContext.destination}</p>
	<p>destination channel interpretation: {audioContext.destination.channelInterpretation}</p>
	<p>destination channel count mode: {audioContext.destination.channelCountMode}</p>
	<p>destination channel count: {audioContext.destination.channelCount}</p>
	<p>destination number of outputs: {audioContext.destination.numberOfOutputs}</p>
	<p>destination number of inputs: {audioContext.destination.numberOfInputs}</p>
	<button on:click={makeNoise}>Make Noise</button>
{/if}

{#if makingNoise}
	<button on:click={stopNoise}>Stop Noise</button>
	<p>some noise!</p>
	<p>OscillatorNode Info</p>
	<p>type: {oscillatorNode.type}</p>
	<p>detune: {oscillatorNode.detune}</p>
	<p>detune value: {oscillatorNode.detune.value}</p>
	<p>detune max value: {oscillatorNode.detune.maxValue}</p>
	<p>detune min value: {oscillatorNode.detune.minValue}</p>
	<p>detune default value: {oscillatorNode.detune.defaultValue}</p>
	<p>detune automation rate: {oscillatorNode.detune.automationRate}</p>
	<p>context: {oscillatorNode.context} basically same data from parent audioContext</p>
	<p>frequency: {oscillatorNode.frequency}</p>
	<p>frequency value: {oscillatorNode.frequency.value}</p>
	<p>frequency automation rate: {oscillatorNode.frequency.automationRate}</p>
	<p>frequency default value: {oscillatorNode.frequency.defaultValue}</p>
	<p>frequency min value: {oscillatorNode.frequency.minValue}</p>
	<p>frequency max value: {oscillatorNode.frequency.maxValue}</p>
	<p>channel count: {oscillatorNode.channelCount}</p>
	<p>number of inputs: {oscillatorNode.numberOfInputs}</p>
	<p>number of outputs: {oscillatorNode.numberOfOutputs}</p>
	<p>channel count mode: {oscillatorNode.channelCountMode}</p>
	<p>channel interpretation {oscillatorNode.channelInterpretation}</p>
{/if}
