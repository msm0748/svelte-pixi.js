<script lang="ts">
	import { onMount } from 'svelte';
	import { Application, Graphics, Container } from 'pixi.js';

	let app: Application;
	let pixiContainer: HTMLDivElement;
	let currentRect: Container | null = null;
	let isDrawing = false;
	let startPos = { x: 0, y: 0 };

	// 리사이즈 핸들 크기
	const HANDLE_SIZE = 10;

	interface BoxContainer extends Container {
		mainRect?: Graphics;
		handles?: Graphics[];
		updateHandlePositions?: () => void;
	}

	onMount(async () => {
		app = new Application();
		await app.init({ width: 800, height: 600, backgroundColor: 0xffffff });
		pixiContainer.appendChild(app.canvas);

		app.stage.eventMode = 'static';
		app.stage.hitArea = app.screen;

		app.stage.on('pointerdown', (event) => {
			isDrawing = true;
			startPos = { x: event.global.x, y: event.global.y };

			// 새로운 사각형 컨테이너 생성
			const boxContainer: BoxContainer = new Container();
			currentRect = boxContainer;

			// 메인 사각형 생성
			const rect = new Graphics()
				.rect(startPos.x, startPos.y, 0, 0)
				.fill({ color: 0x0000ff, alpha: 0.5 })
				.stroke({ width: 2, color: 0x000000 });

			boxContainer.mainRect = rect;
			boxContainer.addChild(rect);
			app.stage.addChild(boxContainer);
		});

		app.stage.on('pointermove', (event) => {
			if (isDrawing && currentRect?.mainRect) {
				const currentPos = { x: event.global.x, y: event.global.y };
				const width = currentPos.x - startPos.x;
				const height = currentPos.y - startPos.y;

				currentRect.mainRect.clear();

				if (width >= 0 && height >= 0) {
					currentRect.mainRect
						.rect(startPos.x, startPos.y, width, height)
						.fill({ color: 0x0000ff, alpha: 0.5 })
						.stroke({ width: 2, color: 0x000000 });
				} else if (width < 0 && height >= 0) {
					currentRect.mainRect
						.rect(currentPos.x, startPos.y, Math.abs(width), height)
						.fill({ color: 0x0000ff, alpha: 0.5 })
						.stroke({ width: 2, color: 0x000000 });
				} else if (width >= 0 && height < 0) {
					currentRect.mainRect
						.rect(startPos.x, currentPos.y, width, Math.abs(height))
						.fill({ color: 0x0000ff, alpha: 0.5 })
						.stroke({ width: 2, color: 0x000000 });
				} else {
					currentRect.mainRect
						.rect(currentPos.x, currentPos.y, Math.abs(width), Math.abs(height))
						.fill({ color: 0x0000ff, alpha: 0.5 })
						.stroke({ width: 2, color: 0x000000 });
				}
			}
		});

		app.stage.on('pointerup', () => {
			isDrawing = false;
			if (currentRect?.mainRect) {
				addResizeHandles(currentRect);
				makeDraggable(currentRect);
				currentRect = null;
			}
		});

		app.stage.on('pointerupoutside', () => {
			isDrawing = false;
			if (currentRect) {
				app.stage.removeChild(currentRect);
				currentRect = null;
			}
		});
	});

	function addResizeHandles(boxContainer: BoxContainer) {
		const rect = boxContainer.mainRect!;
		const bounds = rect.getBounds();
		const handles: Graphics[] = [];

		// 8개의 리사이즈 핸들 생성 (모서리 4개 + 변 중앙 4개)
		const handlePositions = [
			{ x: bounds.x, y: bounds.y }, // 좌상
			{ x: bounds.x + bounds.width / 2, y: bounds.y }, // 상중앙
			{ x: bounds.x + bounds.width, y: bounds.y }, // 우상
			{ x: bounds.x + bounds.width, y: bounds.y + bounds.height / 2 }, // 우중앙
			{ x: bounds.x + bounds.width, y: bounds.y + bounds.height }, // 우하
			{ x: bounds.x + bounds.width / 2, y: bounds.y + bounds.height }, // 하중앙
			{ x: bounds.x, y: bounds.y + bounds.height }, // 좌하
			{ x: bounds.x, y: bounds.y + bounds.height / 2 } // 좌중앙
		];

		handlePositions.forEach((pos, index) => {
			const handle = new Graphics()
				.rect(pos.x - HANDLE_SIZE / 2, pos.y - HANDLE_SIZE / 2, HANDLE_SIZE, HANDLE_SIZE)
				.fill({ color: 0xffffff })
				.stroke({ width: 1, color: 0x000000 });

			handle.eventMode = 'static';
			handle.cursor = getCursorStyle(index);

			handle.on('pointerdown', (event) => {
				event.stopPropagation();
				startResize(boxContainer, handle, index, event);
			});

			handles.push(handle);
			boxContainer.addChild(handle);
		});

		boxContainer.handles = handles;

		// 핸들 위치 업데이트 함수
		boxContainer.updateHandlePositions = () => {
			const newBounds = rect.getBounds();
			const positions = [
				{ x: newBounds.x, y: newBounds.y },
				{ x: newBounds.x + newBounds.width / 2, y: newBounds.y },
				{ x: newBounds.x + newBounds.width, y: newBounds.y },
				{ x: newBounds.x + newBounds.width, y: newBounds.y + newBounds.height / 2 },
				{ x: newBounds.x + newBounds.width, y: newBounds.y + newBounds.height },
				{ x: newBounds.x + newBounds.width / 2, y: newBounds.y + newBounds.height },
				{ x: newBounds.x, y: newBounds.y + newBounds.height },
				{ x: newBounds.x, y: newBounds.y + newBounds.height / 2 }
			];

			handles.forEach((handle, index) => {
				handle.position.set(
					positions[index].x - HANDLE_SIZE / 2,
					positions[index].y - HANDLE_SIZE / 2
				);
			});
		};
	}

	function getCursorStyle(handleIndex: number): string {
		const cursorStyles = [
			'nw-resize',
			'n-resize',
			'ne-resize',
			'e-resize',
			'se-resize',
			's-resize',
			'sw-resize',
			'w-resize'
		];
		return cursorStyles[handleIndex];
	}

	function startResize(
		boxContainer: BoxContainer,
		handle: Graphics,
		handleIndex: number,
		event: any
	) {
		const rect = boxContainer.mainRect!;
		const startBounds = rect.getBounds();
		const startPoint = { x: event.global.x, y: event.global.y };
		const originalSize = { width: startBounds.width, height: startBounds.height };
		const originalPos = { x: startBounds.x, y: startBounds.y };

		const onMove = (moveEvent: any) => {
			const dx = moveEvent.global.x - startPoint.x;
			const dy = moveEvent.global.y - startPoint.y;

			let newWidth = originalSize.width;
			let newHeight = originalSize.height;
			let newX = originalPos.x;
			let newY = originalPos.y;

			// 핸들 위치에 따른 리사이즈 로직
			switch (handleIndex) {
				case 0: // 좌상
					newWidth = originalSize.width - dx;
					newHeight = originalSize.height - dy;
					newX = originalPos.x + dx;
					newY = originalPos.y + dy;
					break;
				case 2: // 우상
					newWidth = originalSize.width + dx;
					newHeight = originalSize.height - dy;
					newY = originalPos.y + dy;
					break;
				case 4: // 우하
					newWidth = originalSize.width + dx;
					newHeight = originalSize.height + dy;
					break;
				case 6: // 좌하
					newWidth = originalSize.width - dx;
					newHeight = originalSize.height + dy;
					newX = originalPos.x + dx;
					break;
				// 추가적인 핸들 케이스들...
			}

			// 최소 크기 제한
			newWidth = Math.max(20, newWidth);
			newHeight = Math.max(20, newHeight);

			// 사각형 업데이트
			rect
				.clear()
				.rect(newX, newY, newWidth, newHeight)
				.fill({ color: 0x0000ff, alpha: 0.5 })
				.stroke({ width: 2, color: 0x000000 });

			// 핸들 위치 업데이트
			boxContainer.updateHandlePositions?.();
		};

		const onEnd = () => {
			app.stage.off('pointermove', onMove);
			app.stage.off('pointerup', onEnd);
			app.stage.off('pointerupoutside', onEnd);
		};

		app.stage.on('pointermove', onMove);
		app.stage.on('pointerup', onEnd);
		app.stage.on('pointerupoutside', onEnd);
	}

	let dragTarget: Container | null = null;
	let dragOffset = { x: 0, y: 0 };

	function makeDraggable(boxContainer: BoxContainer) {
		boxContainer.eventMode = 'static';
		boxContainer.cursor = 'move';

		boxContainer.on('pointerdown', (event) => {
			event.stopPropagation();
			dragTarget = boxContainer;
			const globalPosition = boxContainer.toGlobal({ x: 0, y: 0 });
			dragOffset.x = globalPosition.x - event.global.x;
			dragOffset.y = globalPosition.y - event.global.y;
		});

		app.stage.on('pointermove', (event) => {
			if (dragTarget === boxContainer) {
				const newPosition = {
					x: event.global.x + dragOffset.x,
					y: event.global.y + dragOffset.y
				};
				boxContainer.position.set(newPosition.x, newPosition.y);
			}
		});

		boxContainer.on('pointerup', () => {
			dragTarget = null;
		});

		boxContainer.on('pointerupoutside', () => {
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
