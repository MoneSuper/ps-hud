<script>
	import Color from './color'
	import { onMount } from 'svelte'
	import { getDimension } from './dimensions'
	import { relativePosition } from './event'
	import touchToMouse from './touch'
	export let color = Color.hex('#00ff00')
	export let dimensionX = 'hsl.l'
	export let dimensionY = 'hsl.s'
	export let detailX = 100
	export let detailY = 100
	export let width = null
	export let height = null
	let canvas
	let ctx
	let cWidth
	let cHeight
	onMount(() => {
		ctx = canvas.getContext('2d')
		touchToMouse(canvas)
	})
	$: dimX = getDimension(dimensionX)
	$: dimY = getDimension(dimensionY)
	$: valueX = color.get(dimX.scale, dimX.dim) * dimX.data.scale
	$: valueY = color.get(dimY.scale, dimY.dim) * dimY.data.scale
	$: sliderPosX = cWidth * (valueX - dimX.data.extent[0]) / (dimX.data.extent[1] - dimX.data.extent[0])
	$: sliderPosY = cHeight - cHeight * (valueY - dimY.data.extent[0]) / (dimY.data.extent[1] - dimY.data.extent[0])
	$: {
		if (ctx) {
			ctx.clearRect(0, 0, cWidth, cHeight)
			ctx.imageSmoothingEnabled = false
			let dX = Math.min(detailX, cWidth - 2)
			let dY = Math.min(detailY, cHeight - 2)
			let xStep = (cWidth - 2) / dX
			let yStep = (cHeight - 2) / dY
			let rangeX = dimX.data.extent[1] - dimX.data.extent[0]
			let rangeY = dimY.data.extent[1] - dimY.data.extent[0]
			const colBase = color.to(dimY.scale)
			for (let y = 0; y <= dY; y++) {
				const vY = (y / detailY * rangeY + dimY.data.extent[0]) / dimY.data.scale
				const colY = colBase.alter(dimY.scale, dimY.dim, vY)
				for(let x = 0; x <= dX; x++) {
					const vX = (x / detailX * rangeX + dimX.data.extent[0]) / dimX.data.scale
					const colYX = colY.alter(dimX.scale, dimX.dim, vX)
					ctx.fillStyle = colYX.toHex()
					ctx.fillRect(Math.round(x * xStep), Math.round((cHeight - 2) - y * yStep), Math.ceil(xStep), Math.ceil(yStep))
				}
			}
			ctx.fillStyle = '#ffffff'
			ctx.fillRect(sliderPosX - 2, sliderPosY - 2, 5, 5)
			ctx.fillStyle = '#000000'
			ctx.fillRect(sliderPosX - 1, sliderPosY - 1, 3, 3)
		}
	}
	function onMouse(e) {
		if (e.buttons === 1) {
			const pos = relativePosition(e)
			let x = pos.relativeX
			let y = (cHeight - 2) - pos.relativeY
			let vX = (x / (cWidth - 2)) * (dimX.data.extent[1] - dimX.data.extent[0]) + dimX.data.extent[0]
			if (vX > dimX.data.extent[1]) { vX = dimX.data.extent[1] }
			if (vX < dimX.data.extent[0]) { vX = dimX.data.extent[0] }
			let vY = (y / (cHeight - 2)) * (dimY.data.extent[1] - dimY.data.extent[0]) + dimY.data.extent[0]
			if (vY > dimY.data.extent[1]) { vY = dimY.data.extent[1] }
			if (vY < dimY.data.extent[0]) { vY = dimY.data.extent[0] }
			const colY = color.alter(dimY.scale, dimY.dim, vY / dimY.data.scale)
			color = colY.alter(dimX.scale, dimX.dim, vX / dimX.data.scale)
		}
	}
</script>

<div class='matrix' bind:clientWidth={cWidth} bind:clientHeight={cHeight}>
	<canvas bind:this={canvas} width={width} height={height} on:mousedown={onMouse} on:mousemove={onMouse}>
</div>

<style>
	.matrix, canvas {
		display: inline-block;
		padding: 0;
		margin: 0;
		line-height: 0;
	}
	canvas {
		border: 1px solid #666;
		border-radius: 5px;
		cursor: crosshair;
	}
</style>