<script>
import { onMount } from 'svelte';
import p5 from 'p5';
import ml5 from 'ml5'; 

let canvas
let canvasContainer = $state()
let video
let detector
let detections = $state([]);
let confidence = $state([]);

onMount(async() => {
// onMount(() => {
 	// const ml5 = import('ml5');
  // Initialize the video capture

	new p5((p)=>{
		p.setup = async ()=>{
			canvas = p.createCanvas(740,480)
			canvas.elt.classList.add('canvas');
			video = await p.createCapture(p.VIDEO)
			canvas.flipped = true;
			video.size(740,480)
			video.hide();

			detector = await ml5.objectDetector("cocossd", modelIFReady);
			
		}
		p.draw = ()=>{
			let smoothed = {}
			p.image(video,0,0)
			for(let i = 0; i < detections.length; i++){
				let object = detections[i];
				let id = object.label + i; // rough identity per slot

				// Initialize smoothed position if new
				if (!smoothed[id]) {
					smoothed[id] = { x: object.x, y: object.y, w: object.width, h: object.height };
				}

				// Lerp toward target
				let t = 1; // 0 = no movement, 1 = instant snap — tune this
				smoothed[id].x = p.lerp(smoothed[id].x, object.x, t);
				smoothed[id].y = p.lerp(smoothed[id].y, object.y, t);
				smoothed[id].w = p.lerp(smoothed[id].w, object.width, t);
				smoothed[id].h = p.lerp(smoothed[id].h, object.height, t);

				let text = `${object.label} ${object.confidence.toFixed(2)}`
				let textWidth = p.textWidth(text) + 3;
				let textHeight = 24; // approximate height for text size 24
				let hue = i / detections.length * 120 * object.confidence; // green to red based on confidence
				p.colorMode(p.HSL, 100);

				p.stroke(hue, 100, 50);
				p.strokeWeight(4);
				p.noFill();
				p.rect(object.x,object.y,object.width,object.height)
				
				p.stroke(hue, 100, 50);
				p.noFill();
				p.rect(object.x, object.y - textHeight - 5, textWidth, textHeight);
				
				p.noStroke();
				p.fill(255);
				p.textSize(textHeight);
				p.text(text,object.x, object.y - 5)

			}		
			
		}
		function modelIFReady(){
			console.log('model ready')
			detector.detect(video,yourDetections)
		}

		function yourDetections(error,results){
			if(error){
				console.log(error)
			}
			detections = results
			confidence = detections.confidence
			
			setTimeout(() => {
				detector.detect(video, yourDetections);
			}, 400);
		}
	},canvasContainer)	

	return () => {
		// Clean up resources if needed
		if (video) {
			video.remove();
		}
		if (canvas) {
			canvas.remove();
		}
	
	};
	
});
</script>

<div class="label-container">
	<div>
		<h1> p5 ml5 object detection </h1>
	</div>
	<p>{detections.length} objects detected</p>
	
</div>

<div bind:this={canvasContainer}>
	<video id="video"></video>
	<!-- <canvas id="canvas" class="canvas" bind:this={canvas} style="position: fixed; inset: 0; background: transparent;"></canvas> -->

</div>

<div class="detected">
	<ul >
	{#each detections as detection}
		<li>- {detection.label} detected ({detection.confidence.toFixed(2)})</li>
	{/each}
	</ul>
</div>

<style>

/* * { */
	/* box-sizing: border-box; */
	/* font-family: "Geist", sans-serif;
	font-optical-sizing: auto;
	font-style: normal; */
/* } */
.label-container {
	font-family: "Geist Pixel", sans-serif;
	position: relative;
	display:flex;
	justify-content: space-between;
	width: 100%;
	min-height: 4rem;
	max-height: fit-content;
	font-size: 1.25rem;
	/* font-family: geist; */
	/* border: solid yellow; */
}

:global(div:has(.canvas)) {
	display: flex;
	justify-content: center;
	width: 100%;
}
:global(.canvas) {
	border: solid blue;
	border-radius: 5px;
	width: 100% !important;
	max-width: 90%;
	align-self: center !important;
	height: 2rem ;
}

#video {
	display: none;
}

.detected {
	position: relative;
	display: flex;
	min-height: 5rem;
	max-height: fit-content;
	width: 100%;
	margin-top: 2%;
	font-size: clamp(1rem, 4vh, 3rem);
	overflow-y: auto !important;
}

</style>