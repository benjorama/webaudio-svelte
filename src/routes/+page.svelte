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

	/** @type {StereoPannerNode} */
	let stereoPannerNode;

	/** @type {BiquadFilterNode} */
	let biquadFilterNode;

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
			//audioContext = undefined;
		}
	}

	async function makeNoise() {
		if (audioContext) {
			oscillatorNode = audioContext.createOscillator();
			gainNode = audioContext.createGain();
			stereoPannerNode = audioContext.createStereoPanner();
			biquadFilterNode = audioContext.createBiquadFilter();

			oscillatorNode.connect(gainNode);
			gainNode.connect(stereoPannerNode);
			stereoPannerNode.connect(biquadFilterNode);
			biquadFilterNode.connect(audioContext.destination);

			gainNode.gain.value = 0.1;
			stereoPannerNode.pan.value = 0.0;
			oscillatorNode.type = 'sine';
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

	/**
	 *@param {Event} event
	 */
	function selectOscillator(event) {
		let oscillatorInput = /**@type{HTMLSelectElement}*/ (event.currentTarget);
		oscillatorNode.type = /** @type {OscillatorType} */ (oscillatorInput.value);
	}

	/**
	 *@param {Event} event
	 */
	function adjustGain(event) {
		let gainInput = /**@type {HTMLInputElement}*/ (event.currentTarget);
		gainNode.gain.setValueAtTime(parseFloat(gainInput.value), audioContextCurrentTime + 0.01);
	}

	/**
	 *@param {Event} event
	 */
	function adjustPan(event) {
		let panInput = /**@type {HTMLInputElement}*/ (event.currentTarget);
		stereoPannerNode.pan.setValueAtTime(parseFloat(panInput.value), audioContextCurrentTime + 0.01);
	}

	/**
	 *@param {Event} event
	 */
	function selectFilter(event) {
		let filterInput = /**@type{HTMLSelectElement}*/ (event.currentTarget);
		biquadFilterNode.type = /** @type {BiquadFilterType} */ (filterInput.value);
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

			<div>
				<label for="osc-type">Select oscillator</label>
				<select on:change={selectOscillator} name="osc-type" id="osc-type">
					<option value="sine">sine</option>
					<option value="square">square</option>
					<option value="sawtooth">sawtooth</option>
					<option value="triangle">triangle</option>
				</select>
			</div>

			<div>
				<input
					on:input={adjustGain}
					type="range"
					id="volume"
					name="volume"
					min="0"
					max="1"
					step="0.01"
					value="0.1"
				/>
				<label for="volume">Volume</label>
			</div>

			<div>
				<input
					on:input={adjustPan}
					type="range"
					id="pan"
					name="pan"
					min="-1"
					max="1"
					step="0.01"
					value="0.0"
				/>
				<label for="pan">Pan</label>
			</div>

			<div>
				<label for="filter-type">Select Filter</label>
				<select on:change={selectFilter} name="filter-type" id="filter-type">
					<option value="lowpass">lowpass</option>
					<option value="highpass">highpass</option>
					<option value="bandpass">bandpass</option>
					<option value="lowshelf">lowshelf</option>
					<option value="highshelf">highshelf</option>
					<option value="peaking">peaking</option>
					<option value="notch">notch</option>
					<option value="allpass">allpass</option>
				</select>
			</div>
			<p>type: {oscillatorNode.type}</p>
			<p>frequency value: {oscillatorNode.frequency.value}</p>
		{/if}
	</div>
	<p>state: {audioContextState}</p>
	<p>current time: {audioContextCurrentTime}</p>
	<p>sample rate: {sampleRate}</p>
	<p>base latency: {baseLatency}</p>
	<p>output latency: {outputLatency}</p>
{/if}
