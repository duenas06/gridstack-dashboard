<template>
    <div
        class="flex flex-col h-screen bg-gray-800 align-middle justify-items-center">
        <div class="flex flex-row-reverse p-4 gap-4 text-white">
            <button @click="toggleEditMode"
                class="px-4 py-2 border border-slate-600 hover:bg-slate-600 rounded-md text-white">
                {{ editMode ? 'Disable' : 'Enable' }} Edit Mode
            </button>
            <button @click="addWidget"
                class="px-4 py-2 border border-slate-600 hover:bg-slate-600 rounded-md text-white">
                Add Widget
            </button>
        </div>
        <div>
            <div class="grid-stack" ref="gridContainer">
                <div v-for="widget in widgets" :key="widget.id"
                    class="grid-stack-item cursor-pointer" :gs-id="widget.id"
                    :gs-x="widget.grid.x" :gs-y="widget.grid.y"
                    :gs-w="widget.grid.w" :gs-h="widget.grid.h">
                    <div
                        class="grid-stack-item-content flex flex-col gap-2 justify-center items-center bg-slate-600 hover:bg-gray-800 text-white">
                        <label>{{ widget.title }}</label>
                        <button v-if="editMode" @click="removeWidget(widget.id)"
                            class="ml-1 text-center py-1 px-2 rounded-md bg-red-500">Remove</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { onMounted, ref, watch } from 'vue';
import 'gridstack/dist/gridstack.min.css';
import { GridStack } from 'gridstack';

const widgets = ref([
    { id: 1, title: 'First Widget', grid: { x: 0, y: 0, w: 2, h: 2 } },
    { id: 2, title: 'Second Widget', grid: { x: 2, y: 0, w: 2, h: 1 } },
    { id: 3, title: 'Third Widget', grid: { x: 0, y: 2, w: 2, h: 1 } },
    { id: 4, title: 'Fourth Widget', grid: { x: 2, y: 2, w: 1, h: 2 } },
    { id: 5, title: 'Fifth Widget', grid: { x: 3, y: 2, w: 1, h: 2 } },
]);

let grid;
const editMode = ref(true);

onMounted(() => {
    grid = GridStack.init({
        alwaysShowResizeHandle: 'mobile'
    });

    grid.on('change', (event, items) => {
        items.forEach(item => {
            const widget = widgets.value.find(w => w.id === Number(item.id));
            if (widget) {
                widget.grid.x = item.x;
                widget.grid.y = item.y;
                widget.grid.w = item.w;
                widget.grid.h = item.h;
            }
        });
    });

    watch(widgets, (newWidgets) => {
        newWidgets.forEach(widget => {
            const element = grid.el.querySelector(`[gs-id='${widget.id}']`);
            if (!element) {
                const widgetElement = document.createElement('div');
                widgetElement.className = 'grid-stack-item cursor-pointer';
                widgetElement.setAttribute('key', widget.id);
                widgetElement.setAttribute('gs-id', widget.id);
                widgetElement.setAttribute('gs-x', widget.grid.x);
                widgetElement.setAttribute('gs-y', widget.grid.y);
                widgetElement.setAttribute('gs-w', widget.grid.w);
                widgetElement.setAttribute('gs-h', widget.grid.h);

                const widgetContent = document.createElement('div');
                widgetContent.className = 'grid-stack-item-content flex flex-col gap-2 justify-center items-center bg-slate-600 hover:bg-gray-800 text-white';
                widgetContent.innerHTML = `
                    <label>${widget.title}</label>
                    <button class="ml-1 text-center py-1 px-2 rounded-md bg-red-500">Remove</button>
                `;

                // Attach event listener for remove button
                widgetContent.querySelector('button').addEventListener('click', () => removeWidget(widget.id));
                widgetElement.appendChild(widgetContent);
                grid.addWidget(widgetElement);
            }
        });

        const widgetIds = new Set(newWidgets.map(widget => widget.id));
        grid.engine.nodes.forEach(node => {
            if (!widgetIds.has(Number(node.el.getAttribute('gs-id')))) {
                grid.removeWidget(node.el);
            }
        });
    }, { deep: true });

    watch(editMode, (newEditMode) => {
        document.querySelectorAll('.grid-stack-item-content button').forEach(button => {
            button.style.display = newEditMode ? 'block' : 'none';
        });
    });
});

const addWidget = () => {
    const newWidget = {
        id: widgets.value.length + 1,
        title: `New Widget ${widgets.value.length + 1}`,
        grid: { x: 0, y: 0, w: 2, h: 1 },
    };
    widgets.value.push(newWidget);
};

const removeWidget = (id) => {
    widgets.value = widgets.value.filter(widget => widget.id !== id);
};

const toggleEditMode = () => {
    editMode.value = !editMode.value;
    grid.setStatic(!editMode.value);
};
</script>

<style scoped>
.grid-stack-item-content {
    display: flex;
    border-radius: 8px;
    box-sizing: border-box;
    align-items: center;
    justify-content: center;
    text-align: center;
}

.grid-stack-item-content button {
    display: none;
}

.grid-stack-item-content button.ml-1.text-center.py-1.px-2.rounded-md.bg-red-500 {
    display: block;
}</style>
