<script lang="ts">
    // Получаем пропсы, задаём дефолты и собираем всё остальное в rest
    let {
        id,
        x = 0,
        y = 0,
        width = 300,
        height = 200,
        title = "Widget",
        Content,
        contentProps = {},
        onmousedown,
        onremove,
        onresizestart,
        ...rest // сюда попадут event‑handler’ы и любые другие пропсы
    } = $props();
    

    function handleRemoveClick(event: MouseEvent) {
        event.stopPropagation();
        if (onremove) {
            onremove();
        }
    }

    function handleResizeMouseDown(event: MouseEvent) {
        event.stopPropagation();
        if (onresizestart) {
            onresizestart(event);
        }
    }
</script>

<div
    {...rest}
    class="widget"
    style={`left:${x}px; top:${y}px; width:${width}px; height:${height}px;`}
>
    <header {onmousedown} role="button" tabindex="0">
        <span class="title">{title}</span>
        <button
            class="remove-btn"
            onclick={handleRemoveClick}
            aria-label="Remove widget">×</button
        >
    </header>
    <div class="body">
        {#if Content}
            <Content {...contentProps}></Content>
        {/if}
    </div>
    <div
        class="resize-handle"
        onmousedown={handleResizeMouseDown}
        role="button"
        tabindex="0"
    ></div>
</div>

<style>
    .widget {
        position: absolute;
        background: #fff;
        border-radius: 8px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        display: flex;
        flex-direction: column;
    }
    header {
        display: flex; /* Use flexbox for layout */
        justify-content: space-between; /* Space title and button */
        align-items: center; /* Vertically align items */
        padding: 0.5rem;
        font-weight: bold;
        border-bottom: 1px solid #eee;
        cursor: grab;
        user-select: none;
    }
    .title {
        flex-grow: 1; /* Allow title to take available space */
        margin-right: 0.5rem; /* Add space between title and button */
    }
    .remove-btn {
        background: none;
        border: none;
        font-size: 1.2rem;
        line-height: 1;
        cursor: pointer;
        padding: 0 0.3rem;
        color: #888;
    }
    .remove-btn:hover {
        color: #f00;
    }
    .body {
        flex: 1;
        overflow: hidden;
        padding: 0.5rem;
        position: relative;
    }
    .resize-handle {
        position: absolute;
        bottom: 0;
        right: 0;
        width: 15px;
        height: 15px;
        background: #ccc;
        border: 1px solid #aaa;
        cursor: nwse-resize;
        border-radius: 0 0 8px 0; /* Match bottom-right corner */
    }
    .resize-handle:hover {
        background: #bbb;
    }
</style>
