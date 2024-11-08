<script lang="ts">
	import { onMount } from 'svelte';
	import { Application, Graphics } from 'pixi.js';

	let app: Application;
	let polygonGraphics: Graphics;
	let selectedPoint: { x: number; y: number } | null = null;

	// 초기 다각형 포인트 설정
	let points = [
		{ x: 100, y: 100 },
		{ x: 200, y: 50 },
		{ x: 300, y: 100 },
		{ x: 250, y: 200 },
		{ x: 150, y: 200 }
	];

	// 다각형을 그리는 함수
	function drawPolygon() {
		polygonGraphics.clear();
		polygonGraphics.strokeStyle(2, 0xff0000);
		polygonGraphics.beginFill(0xffd700, 0.5);

		polygonGraphics.moveTo(points[0].x, points[0].y);
		points.forEach((point) => {
			polygonGraphics.lineTo(point.x, point.y);
		});
		polygonGraphics.closePath();
		polygonGraphics.endFill();
	}

	// 이벤트 핸들러 함수
	function onPointerDown(event) {
		const { x, y } = event.data.global;
		selectedPoint = points.find((point) => Math.hypot(point.x - x, point.y - y) < 10);
	}

	function onPointerMove(event) {
		if (selectedPoint) {
			const { x, y } = event.data.global;
			selectedPoint.x = x;
			selectedPoint.y = y;
			drawPolygon();
		}
	}

	function onPointerUp() {
		selectedPoint = null;
	}

	// Svelte 컴포넌트 마운트 시 PixiJS 초기화
	onMount(async () => {
		app = new Application();
		await app.init({ width: 800, height: 600, backgroundColor: 0xffffff });

		// PixiJS canvas를 페이지에 추가
		console.log(app.canvas, 'test');
		document.getElementById('pixi-container').appendChild(app.canvas);

		// Graphics 객체 생성 및 화면 추가
		const polygonGraphics = new Graphics();
		app.stage.addChild(polygonGraphics);

		// 초기 다각형 그리기
		// drawPolygon();

		// 마우스 이벤트 추가
		app.stage.interactive = true;
		// app.stage.on('pointerdown', onPointerDown);
		app.stage.on('pointerdown', () => console.log('asdf'));
		app.stage.on('pointermove', onPointerMove);
		app.stage.on('pointerup', onPointerUp);
	});
</script>

<div id="pixi-container"></div>
