<script lang="ts">
	import { onMount } from 'svelte';
	import { Application, Graphics, Polygon } from 'pixi.js';

	let app: Application;
	let pixiContainer: HTMLDivElement;
	let canvas: HTMLCanvasElement;
	let selectedPoint: { x: number; y: number } | null = null;

	// 초기 다각형 포인트 설정
	let points = [
		{ x: 100, y: 100 },
		{ x: 200, y: 50 },
		{ x: 300, y: 100 },
		{ x: 250, y: 200 },
		{ x: 150, y: 200 }
	];

	let dragTarget = null;

	function onDragMove(event) {
		if (dragTarget) {
			dragTarget.position.x = 50;
			dragTarget.position.y = 50;

			// dragTarget.parent.toLocal(event.global, null, dragTarget.position);
		}
	}

	function onDragStart() {
		// Store a reference to the data
		// * The reason for this is because of multitouch *
		// * We want to track the movement of this particular touch *
		this.alpha = 0.5;
		dragTarget = this;
		app.stage.on('pointermove', onDragMove);
	}

	function onDragEnd() {
		if (dragTarget) {
			app.stage.off('pointermove', onDragMove);
			dragTarget.alpha = 1;
			dragTarget = null;
		}
	}

	// Svelte 컴포넌트 마운트 시 PixiJS 초기화
	onMount(async () => {
		app = new Application();
		await app.init({ width: 800, height: 600, backgroundColor: 0xffffff });

		// PixiJS canvas를 페이지에 추가
		console.log(app.canvas, 'test');
		pixiContainer.appendChild(app.canvas);

		canvas = app.canvas;

		// Graphics 객체 생성 및 화면 추가
		// const polygon2 = new Polygon([0, 0, 100, 100, 400, 250]);
		// polygon2.closePath = true;

		// const polygonGraphics = new Graphics(polygon2).fill({ alpha: 0.5, color: 0xff0000 });
		const polygonGraphics = new Graphics();
		polygonGraphics.poly([0, 0, 100, 100, 400, 250]).fill({ alpha: 0.5, color: 0xff0000 });

		polygonGraphics.interactive = true;

		polygonGraphics.eventMode = 'static';
		polygonGraphics.cursor = 'pointer';

		polygonGraphics.on('pointerdown', onDragStart, polygonGraphics);

		app.stage.addChild(polygonGraphics);
		// polygonGraphics.drawPolygon

		// 초기 다각형 그리기
		// drawPolygon();

		// 마우스 이벤트 추가
		app.stage.interactive = true;
		// app.stage.on('pointerdown', onPointerDown);
		// app.stage.on('pointerdown', () => console.log('asdf'));
		// app.stage.on('pointermove', onPointerMove);
		// app.stage.on('pointerup', onPointerUp);

		app.stage.eventMode = 'static';
		app.stage.hitArea = app.screen;
		app.stage.on('pointerdown', () => {
			console.log('sdalfkjsdalkfj');
		});
		app.stage.on('pointerup', onDragEnd);
		app.stage.on('pointerupoutside', onDragEnd);
	});
</script>

<div id="pixi-container" bind:this={pixiContainer}></div>
