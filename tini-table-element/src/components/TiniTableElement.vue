<script setup lang="ts">
import { onBeforeUnmount, onMounted, reactive, ref } from 'vue';

const props = defineProps({
    title: {
        type: String,
        default: "haha"
    },
    rows: {
        type: Number,
        default: 0
    },
    cols: {
        type: Number,
        default: 0
    }
});
const isMouseDown = ref(false);
const startCell = ref<HTMLTableCellElement | any>();
const endCell = ref<HTMLTableCellElement | any>();
const tiniTable = ref<HTMLTableElement | any>();
const chosenCells = ref<HTMLTableCellElement[]>([])
// const startRow = ref<number | undefined>();
// const endRow = ref<number | undefined>();

// const startCol = ref<number | undefined>();
// const endCol = ref<number | undefined>();
/**
 * 初始化表格参数
 * @param rows 
 * @param cols 
 * @param content 
 */
const tableInit = <T>(rows: number, cols: number, content: T): T[][] => {
    const res = [];
    for (let i = 0; i < rows; i++) {
        const temp = [];
        for (let j = 0; j < cols; j++) {
            temp.push(content);
        }
        res.push([...temp])
    }
    return res;
}

const tableInfo = reactive(tableInit(props.rows, props.cols, { value: '1', edit: false }));

function mousedown(event: MouseEvent): void {
    startCell.value = event.target as HTMLTableCellElement;
    endCell.value = event.target as HTMLTableCellElement;
    cancelChosen()
    chooseCell();
    isMouseDown.value = true;
}

function mouseover(event: MouseEvent) {
    if (!isMouseDown.value) {
        return;
    }
    endCell.value = event.target as HTMLTableCellElement;
    chooseCell()
}

function mouseup(event: MouseEvent) {
    if (isMouseDown.value) {
        isMouseDown.value = false;
    }
}

function chooseCell(): void {
    let [startRow, endRow] = [
        (startCell.value?.parentNode as HTMLTableRowElement).rowIndex,
        (endCell.value?.parentNode as HTMLTableRowElement).rowIndex
    ].sort();
    let [startCol, endCol] = [
        startCell.value?.cellIndex,
        endCell.value?.cellIndex
    ].sort();
    for (let i = 0; i < props.rows; i++) {
        let rows = tiniTable.value?.rows[i]
        for (let j = 0; j < props.cols; j++) {
            let cell = rows.cells[j];
            if (i >= startRow && i <= endRow && j >= startCol && j <= endCol) {
                cell.style.backgroundColor = 'gray';
            } else {
                cell.style.backgroundColor = '';
            }
        }
    }
}

function cancelChosen() {
    const cells = Array.prototype.slice.call(tiniTable.value?.getElementsByTagName('td'));
    chosenCells.value.forEach(cell => {
        cell.style.backgroundColor = 'white'
    })
    chosenCells.value = []
}

onMounted(() => {
    window.addEventListener('click', cancelChosen)
})
onBeforeUnmount(() => {
    window.removeEventListener('click', cancelChosen)
})

</script>

<template>
    <span>表格的行数：{{ props.rows }}</span>
    <span>表格的列数：{{ props.cols }}</span>
    <div class="menu">
        <span class="menu-item">合并单元格</span>
        <span class="menu-item">新增一行</span>
        <span class="menu-item">新增一列</span>
        <span class="menu-item">删除行</span>
        <span class="menu-item">删除列</span>
    </div>
    <div>

        <table @click.stop ref="tiniTable">
            <tr v-for="row in  tableInfo  ">
                <td @mousedown="mousedown($event)" @mouseover="mouseover($event)" @mouseup="mouseup($event)"
                    v-for="data in  row ">
                    <span v-if="data.edit">{{ data.value }}</span>
                    <textarea v-if="!data.edit"></textarea>
                </td>
            </tr>
        </table>
    </div>
</template>

<style scoped>
table {
    border-collapse: collapse;
    /* 合并单元格边框 */
    width: 100%;
    border: 1px solid black
}

td {
    text-align: left;
    padding: 8px;
    user-select: none;
    border: 1px solid black;
    /* 添加底部横线 */
}

.menu {
    height: 25px;
    width: 400px;
    display: flex;

    border: 1px solid black;
    border-radius: 4%;
}

.menu-item {
    flex: 1;
    border: 1px solid black;
}
</style>
