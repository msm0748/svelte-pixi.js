<script lang="ts">
	import { onMount } from 'svelte';
	import { Application, Graphics } from 'pixi.js';

	let app: Application;
	let pixiContainer: HTMLDivElement;
	let currentRect: Graphics | null = null;
	let isDrawing = false;
	let startPos = { x: 0, y: 0 };

	onMount(async () => {
		app = new Application();
		await app.init({ width: 800, height: 600, backgroundColor: 0xffffff });
		pixiContainer.appendChild(app.canvas);

		// 스테이지 이벤트 설정
		app.stage.eventMode = 'static';
		app.stage.hitArea = app.screen;

		// 드래그 시작
		app.stage.on('pointerdown', (event) => {
			console.log('test');
			isDrawing = true;
			startPos = { x: event.global.x, y: event.global.y };

			// 새로운 사각형 생성
			currentRect = new Graphics()
				.rect(startPos.x, startPos.y, 0, 0) // 초기 사각형 (크기 0)
				.fill({ color: 0x0000ff, alpha: 0.5 })
				.stroke({ width: 2, color: 0x000000 });

			app.stage.addChild(currentRect);
		});

		// 드래그 중
		app.stage.on('pointermove', (event) => {
			if (isDrawing && currentRect) {
				const currentPos = { x: event.global.x, y: event.global.y };

				// 사각형 크기와 위치 계산
				const width = currentPos.x - startPos.x;
				const height = currentPos.y - startPos.y;

				// 사각형 다시 그리기
				currentRect.clear();

				// 음수 너비/높이 처리
				if (width >= 0 && height >= 0) {
					currentRect.rect(startPos.x, startPos.y, width, height);
				} else if (width < 0 && height >= 0) {
					currentRect.rect(currentPos.x, startPos.y, Math.abs(width), height);
				} else if (width >= 0 && height < 0) {
					currentRect.rect(startPos.x, currentPos.y, width, Math.abs(height));
				} else {
					currentRect.rect(currentPos.x, currentPos.y, Math.abs(width), Math.abs(height));
				}

				currentRect.fill({ color: 0x0000ff, alpha: 0.5 }).stroke({ width: 2, color: 0x000000 });
			}
		});

		// 드래그 끝
		app.stage.on('pointerup', () => {
			isDrawing = false;
			if (currentRect) {
				// 사각형을 드래그 가능하게 만들기
				makeDraggable(currentRect);
				currentRect = null;
			}
		});

		// 드래그 취소 (스테이지 밖으로 나갔을 때)
		app.stage.on('pointerupoutside', () => {
			isDrawing = false;
			if (currentRect) {
				app.stage.removeChild(currentRect);
				currentRect = null;
			}
		});
	});

	// 드래그 가능하게 만드는 함수
	let dragTarget: Graphics | null = null;
	let dragOffset = { x: 0, y: 0 };

	function makeDraggable(object: Graphics) {
		object.eventMode = 'static';
		object.cursor = 'pointer';

		object.on('pointerdown', (event) => {
			console.log('ㅁㄴㅇㄹ');
			event.stopPropagation();
			dragTarget = object;
			dragOffset.x = object.position.x - event.global.x;
			dragOffset.y = object.position.y - event.global.y;
		});

		app.stage.on('pointermove', (event) => {
			if (dragTarget === object) {
				object.position.x = event.global.x + dragOffset.x;
				object.position.y = event.global.y + dragOffset.y;
			}
		});

		object.on('pointerup', () => {
			dragTarget = null;
		});

		object.on('pointerupoutside', () => {
			dragTarget = null;
		});
	}
</script>

<div bind:this={pixiContainer}>
	<!-- PixiJS canvas가 여기에 자동으로 추가됩니다 -->
</div>

<style>
	div {
		width: 800px;
		height: 600px;
	}
</style>
