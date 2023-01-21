<script lang="ts">
	// import { M, Motion, useDragControls } from 'svelte-motion';
	import { spring } from 'svelte/motion';
	import { onMount } from 'svelte';

	let viewport: HTMLElement;
	let test: string;

	let coords = spring(
		{ x: 0, y: 0 },
		{
			stiffness: 0.02,
			damping: 0.2
		}
	);

	function updateCoords(diffX: number, diffY: number, multiplier: number = 1) {
		let newX = $coords.x + diffX * multiplier;
		let newY = $coords.y + diffY * multiplier;

		const lowerBoundX = 0;
		const upperBoundX = window.innerWidth;
		const lowerBoundY = 0;
		const upperBoundY = window.innerHeight;

		newX = Math.min(Math.max(newX, lowerBoundX), upperBoundX);
		newY = Math.min(Math.max(newY, lowerBoundY), upperBoundY);
		coords.set({ x: newX, y: newY });
	}

	onMount(() => {
		// for mouse movements
		let mouseController = new AbortController();
		let mouseStartX;
		let mouseEndX;
		let mouseStartY;
		let mouseEndY;

		viewport.addEventListener('mousedown', () => {
			mouseController = new AbortController();
			viewport.addEventListener(
				'mousemove',
				(e) => {
					e.preventDefault();
					updateCoords(e.movementX, e.movementY, 5);
				},
				{ signal: mouseController.signal }
			);
		});

		viewport.addEventListener('mouseup', () => {
			mouseController.abort();
		});

		viewport.addEventListener('mouseout', () => {
			mouseController.abort();
		});

		// for touch movement
		let touchController = new AbortController();
		let currentTouchX: number;
		let currentTouchY: number;

		viewport.addEventListener('touchstart', (e) => {
			currentTouchX = e.targetTouches[0].clientX;
			currentTouchY = e.targetTouches[0].clientY;
		});

		viewport.addEventListener('touchmove', (e) => {
			updateCoords(e.touches[0].clientX - currentTouchX, e.touches[0].clientY - currentTouchY, 5);

			currentTouchX = e.touches[0].clientX;
			currentTouchY = e.touches[0].clientY;
			console.log(e);
		});

		// 	viewport.addEventListener('touchstart', () => {
		// 		touchController = new AbortController();
		// 		viewport.addEventListener(
		// 			'touchmove',
		// 			(e) => {
		// 				e.preventDefault();
		// 				console.log(e);
		// 				// updateCoords(e., e.movementY);
		// 			},
		// 			{ signal: touchController.signal }
		// 		);
		// 	});

		// 	viewport.addEventListener('touchend', () => {
		// 		touchController.abort();
		// 	});
	});
</script>

<div id="viewport" bind:this={viewport}>
	<img
		style="transform: translate({$coords.x}px, {$coords.y}px);"
		src="https://d45frc3hu5unh.cloudfront.net/krists-luhaers-O13vJ2omxCI-unsplash.jpg"
		alt=""
	/>
	<h1>x: {$coords.x}, y: {$coords.y}</h1>
	<h1>{test}</h1>
</div>

<style>
	:global(body) {
		margin: 0;
		overflow: hidden;
	}

	#viewport {
		/* height: calc(50% + 50vh);
		width: calc(50% + 50vw); */
		height: 100svh;
		width: 100%;
		/* background: url('https://images.pexels.com/photos/1591447/pexels-photo-1591447.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2')
			no-repeat;
		background-size: 100%; */
	}

	img {
		height: 20vh;
	}
</style>
