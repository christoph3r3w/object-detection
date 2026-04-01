<script>
import { onMount } from 'svelte';
import {page} from '$app/stores';
import p5 from 'p5';
import ml5 from 'ml5'; 

let canvas
let canvasContainer = $state()
let video
let detector
let detections = $state([]);
let confidence = $state([]);
let errorMessage = $state(null);

onMount(async() => {

 	// const ml5 = import('ml5');
	 try {
        await navigator.mediaDevices.getUserMedia({ video: true });
    } catch (e) {
        if (e.name === 'NotAllowedError') {
            errorMessage = 'Camera permission denied';
        } else if (e.name === 'NotFoundError') {
            errorMessage = 'No camera found';
        } else {
            errorMessage = 'Error accessing camera';
        }
    }
  
  	// Initialize the video capture and object detector
	new p5((p)=>{
		p.setup = async ()=>{
			canvas = p.createCanvas(740,480)
			canvas.elt.classList.add('canvas');
			video = await p.createCapture(p.VIDEO)
			// canvas.flipped = true;
			video.size(740,480)
			video.hide();

			detector = await ml5.objectDetector("cocossd", modelIFReady);
			
		}
		p.draw = ()=>{
			let smoothed = {}
			p.image(video,0,0)
			for(let i = 0; i < detections.length; i++){
				let object = detections[i];
				let id = object.label + i; 

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
				let textHeight = 24; 
				let hue = i / detections.length * 120 * object.confidence; 
				p.colorMode(p.HSL, 100);

				p.stroke(hue, 100, 50);
				p.strokeWeight(4);
				p.noFill();
				p.rect(object.x,object.y,object.width,object.height)
				
				p.fill(hue, 100, 50);
				p.rect(object.x, object.y - textHeight - 2, textWidth, textHeight,2);
				
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
				errorMessage = error;
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

<div bind:this={canvasContainer} width="500px" height="400px" class="canvasContainer" error={errorMessage}>
	<!-- <video id="video"></video> -->
</div>

<div class="detected">
	<ul >
	{#each detections as detection}
		<li>- {detection.label} detected ({detection.confidence.toFixed(2)})</li>
		<li>- {detection.label} detected ({detection.confidence.toFixed(2)})</li>
		<li>- {detection.label} detected ({detection.confidence.toFixed(2)})</li>
	{/each}
	</ul>
</div>

<style>

* {
	font-family: "Geist Pixel", sans-serif;
	box-sizing: border-box;
	transition: all 400ms ease-out !important;
}
.label-container {
	flex: 1 0;
	position: relative;
	display:flex;
	justify-content: space-between;
	width: 100%;
	min-height: 4rem;
	max-height: fit-content;
	max-width: 90%;
	font-size: 1.25rem;
	/* border: solid 1px red; */
}

:global(.canvasContainer) {
	flex: 1 0;
	display: flex;
	justify-content: center;
	width: 100%;

	&::after {
		content:attr(error, "Canvas loading...");
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		font-size: 1.5rem;
		color: #333;
	}
}
:global(.canvas) {
	border: solid blue;
	border-radius: 5px;
	width: 100% !important;
	max-width: 90%;
	align-self: center !important;
	height: 2rem ;	
	z-index: 3;

	@starting-style{
		width: 500px;
		height: 400px;
	}
}

.detected {
	position: relative;
	display: flex;
	min-height: 5rem;
	max-height: 10dvh;
	width: 100%;
	max-width: 90%;
	margin-top: 2%;
	font-size: clamp(1rem, 4vh, 3rem);
	overflow-y: scroll !important;
}

</style>