<script lang="ts">
    import Widget from "./Widget.svelte";
    import ChartA from "./ChartA.svelte";
    import { onDestroy } from "svelte";

    type WidgetDef = {
        id: number;
        x: number;
        y: number;
        width: number;
        height: number;
        title: string;
        content: typeof ChartA;
    };

    let widgets: WidgetDef[] = $state([
        {
            id: 1,
            x: 50,
            y: 80,
            width: 300,
            height: 200,
            title: "📈 Chart A",
            content: ChartA,
        },
        {
            id: 2,
            x: 400,
            y: 120,
            width: 600,
            height: 400,
            title: "📊 Chart B",
            content: ChartA,
        },
    ]);

    const availableWidgets = {
        chartA: { title: "📈 Chart A", content: ChartA },
        chartB: { title: "📊 Chart B", content: ChartA },
        chartC: { title: "📉 Chart C", content: ChartA },
    };
    type AvailableWidgetKey = keyof typeof availableWidgets;

    let draggingId: number | null = null;
    let resizingId: number | null = null; // Add state for resizing
    let initialMouseX = 0; // Store initial mouse position for resize
    let initialMouseY = 0;
    let initialWidth = 0; // Store initial widget dimensions for resize
    let initialHeight = 0;
    let offsetX = 0;
    let offsetY = 0;
    let nextWidgetId = $state(3);
    const gridSize = 20;
    const minWidth = 100; // Minimum widget width
    const minHeight = 80; // Minimum widget height

    function isColliding(
        a: { x: number; y: number; width: number; height: number },
        b: { x: number; y: number; width: number; height: number },
    ) {
        return (
            a.x < b.x + b.width &&
            a.x + a.width > b.x &&
            a.y < b.y + b.height &&
            a.y + a.height > b.y
        );
    }

    function hasCollision(id: number, x: number, y: number) {
        const w = widgets.find((w) => w.id === id)!;
        const rectA = { x, y, width: w.width, height: w.height };
        return widgets.some((other) => {
            if (other.id === id) return false;
            const rectB = {
                x: other.x,
                y: other.y,
                width: other.width,
                height: other.height,
            };
            return isColliding(rectA, rectB);
        });
    }
    
    function findFreePosition(width: number, height: number) {
    const maxCols = Math.floor(window.innerWidth / gridSize);
    const maxRows = Math.floor(window.innerHeight / gridSize);

    for (let row = 0; row < maxRows; row++) {
        for (let col = 0; col < maxCols; col++) {
            const x = col * gridSize;
            const y = row * gridSize;

            const rectA = { x, y, width, height };
            const collision = widgets.some(other => {
                const rectB = {
                    x: other.x,
                    y: other.y,
                    width: other.width,
                    height: other.height,
                };
                return isColliding(rectA, rectB);
            });

            if (!collision) {
                return { x, y };
            }
        }
    }
    // если не нашлось — возвращаем дефолт
    return { x: 50, y: 50 };
}


    function startDrag(e: MouseEvent, id: number) {
        draggingId = id;
        const w = widgets.find((w) => w.id === id)!;
        offsetX = e.clientX - w.x;
        offsetY = e.clientY - w.y;

        window.addEventListener("mousemove", onDrag);
        window.addEventListener("mouseup", stopDrag);
    }

    function onDrag(e: MouseEvent) {
        if (draggingId !== null) {
            const w = widgets.find((w) => w.id === draggingId);
            if (w) {
                const newX = Math.max(0, e.clientX - offsetX);
                const newY = Math.max(0, e.clientY - offsetY);

                //const snappedX = Math.round(newX / gridSize) * gridSize;
                //const snappedY = Math.round(newY / gridSize) * gridSize;

                if (!hasCollision(w.id, newX, newY)) {
                    w.x = newX;
                    w.y = newY;
                }
            }
        }
    }

    function stopDrag() {
        if (draggingId !== null) {
            // Remove listeners from the window when drag stops
            const w = widgets.find((w) => w.id === draggingId);
            if (w) {
                const snappedX = Math.round(w.x / gridSize) * gridSize;
                const snappedY = Math.round(w.y / gridSize) * gridSize;
                w.x = snappedX;
                w.y = snappedY;
            }
            window.removeEventListener("mousemove", onDrag);
            window.removeEventListener("mouseup", stopDrag);
            draggingId = null;
        }
    }

    function startResize(e: MouseEvent, id: number) {
        resizingId = id;
        const w = widgets.find((w) => w.id === id)!;
        initialMouseX = e.clientX;
        initialMouseY = e.clientY;
        initialWidth = w.width;
        initialHeight = w.height;

        window.addEventListener("mousemove", onResize);
        window.addEventListener("mouseup", stopResize);
    }

    function onResize(e: MouseEvent) {
        if (resizingId !== null) {
            const w = widgets.find((w) => w.id === resizingId);
            if (w) {
                const dx = e.clientX - initialMouseX;
                const dy = e.clientY - initialMouseY;

                // Calculate new dimensions, respecting minimum size
                const newWidth = Math.max(minWidth, initialWidth + dx);
                const newHeight = Math.max(minHeight, initialHeight + dy);

                if (!hasCollision(w.id, w.x, w.y)) {
                    w.width = newWidth;
                    w.height = newHeight;
                }
            }
        }
    }

    function stopResize() {
        if (resizingId !== null) {
            const w = widgets.find((w) => w.id === resizingId);
            if (w) {
                // Snap dimensions to grid
                w.width = Math.round(w.width / gridSize) * gridSize;
                w.height = Math.round(w.height / gridSize) * gridSize;
                // Ensure minimum size after snapping
                w.width = Math.max(minWidth, w.width);
                w.height = Math.max(minHeight, w.height);
            }
            window.removeEventListener("mousemove", onResize);
            window.removeEventListener("mouseup", stopResize);
            resizingId = null;
        }
    }

    function addWidget(event: Event) {
        const selectElement = event.target as HTMLSelectElement;
        const selectedWidget = selectElement.value as AvailableWidgetKey;

        if (selectElement && availableWidgets[selectedWidget]) {
            const widgetToAdd = availableWidgets[selectedWidget];
            const newWidget: WidgetDef = {
                id: nextWidgetId,
                x: 0,
                y: 0,
                width: 300,
                height: 200,
                title: widgetToAdd.title,
                content: widgetToAdd.content,
            };
            const { x, y } = findFreePosition(newWidget.width,newWidget.height);
            console.log(x,y);
            newWidget.x = x;
            newWidget.y = y;
            widgets.push(newWidget);
            nextWidgetId++;
        }

        selectElement.value = "";
    }
    function removeWidget(id: number) {
        widgets = widgets.filter((widget) => widget.id !== id);
    }

    onDestroy(() => {
        window.removeEventListener("mousemove", onDrag);
        window.removeEventListener("mouseup", stopDrag);
    });
</script>

<div class="controls">
    <select onchange={addWidget}>
        <option value="" disabled selected>Add Widget...</option>
        {#each Object.entries(availableWidgets) as [key, widgetInfo]}
            <option value={key}>{widgetInfo.title}</option>
        {/each}
    </select>
</div>

<div class="dashboard" role="application">
    {#each widgets as widget (widget.id)}
        <Widget
            id={widget.id}
            x={widget.x}
            y={widget.y}
            width={widget.width}
            height={widget.height}
            title={widget.title}
            content={widget.content}
            onmousedown={(e: MouseEvent) => startDrag(e, widget.id)}
            onremove={(e: MouseEvent) => removeWidget(widget.id)}
            onresizestart={(e: MouseEvent) => startResize(e, widget.id)}
        />
    {/each}
</div>

<style>
    .controls {
        padding: 10px;
        background-color: #e0e0e0;
        border-bottom: 1px solid #ccc;
    }
    .controls select {
        padding: 5px;
    }
    .dashboard {
        position: relative;
        width: 100vw;
        height: 100vh;
        background: #f0f0f0;
        overflow: hidden;
    }
</style>
