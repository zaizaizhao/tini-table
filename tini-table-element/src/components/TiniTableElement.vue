<script setup lang="ts">
import { onBeforeUnmount, onMounted, reactive, ref } from "vue";

const props = defineProps({
  title: {
    type: String,
    default: "haha",
  },
  rows: {
    type: Number,
    default: 0,
  },
  cols: {
    type: Number,
    default: 0,
  },
});
const isMouseDown = ref(false);
const startCell = ref<HTMLTableCellElement | any>();
const endCell = ref<HTMLTableCellElement | any>();
const tiniTable = ref<HTMLTableElement | any>();
const chosenCells = ref<HTMLTableCellElement[]>([]);
const editedCell = ref<CellType>();

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
      temp.push(JSON.parse(JSON.stringify(content)));
    }
    res.push([...temp]);
  }
  return res;
};

const tableInfo = reactive(
  tableInit(props.rows, props.cols, { value: "1", edit: false })
);

function mousedown(event: MouseEvent, data: CellType): void {
  if (editedCell.value && editedCell.value != data) {
    editedCell.value.edit = false;
  }
  let node;
  if((event.target as HTMLElement).tagName  != "TD"){
   node = (event.target as HTMLElement).parentNode
  }else{
    node = event.target;
  }
  startCell.value =node as HTMLTableCellElement;
  endCell.value = node as HTMLTableCellElement;
  cancelChosen();
  chooseCell();
  isMouseDown.value = true;
}

function mouseover(event: MouseEvent) {
  if (!isMouseDown.value) {
    return;
  }
  let node;
  if((event.target as HTMLElement).tagName  != "TD"){
   node = (event.target as HTMLElement).parentNode
  }else{
    node = event.target;
  }
  endCell.value = node as HTMLTableCellElement;
  chooseCell();
}

function mouseup(event: MouseEvent) {
  if (isMouseDown.value) {
    isMouseDown.value = false;
  }
}

function chooseCell(): void {
  const {
    startRow,
    endRow,
    startCol,
    endCol,
  } =CalculateStartAndEnd (startCell.value,endCell.value)
  for (let i = 0; i < props.rows; i++) {
    let rows = tiniTable.value?.rows[i];
    for (let j = 0; j < props.cols; j++) {
      let cell = rows.cells[j];
      if (i >= startRow && i <= endRow && j >= startCol && j <= endCol) {
        cell.style.backgroundColor = "gray";
      } else {
        cell.style.backgroundColor = "";
      }
    }
  }
}

//! 计算起止节点
function CalculateStartAndEnd(startCell:HTMLTableCellElement,endCell:HTMLTableCellElement){
  let [startRow, endRow] = [
    (startCell.parentNode as HTMLTableRowElement).rowIndex,
    (endCell.parentNode as HTMLTableRowElement).rowIndex,
  ].sort();
  let [startCol, endCol] = [
    startCell.cellIndex,
    endCell.cellIndex,
  ].sort();
  return {
    startRow,
    endRow,
    startCol,
    endCol,
  }
}

function cancelChosen() {
  const cells = Array.prototype.slice.call(
    tiniTable.value?.getElementsByTagName("td")
  );
  chosenCells.value.forEach((cell) => {
    cell.style.backgroundColor = "white";
  });
  chosenCells.value = [];
}

//! 开始编辑单元格
function startEdit(value: CellType) {
  console.log(value);
if(value.edit){
  return
}
  editedCell.value = value;
  value.edit = true;
  setTimeout(() => {
    const targetEle = document.querySelector(".edit-textarea") as HTMLElement;
    targetEle.innerText = value.value;
    set_focus(targetEle);
    targetEle.addEventListener("blur", () => {
      console.log("blur",editedCell.value);
      editedCell.value!.value = targetEle.innerText;
      value.value= targetEle.innerText
    });
  }, 10);
}

function set_focus(el: HTMLElement) {
  console.log(el);
  el.contentEditable = 'true'
  //	el.focus();
  //创建一个range范围对象
  const range = document.createRange();
  //用于设置 Range，使其包含一个 Node的内容。
  range.selectNodeContents(el);
  //将包含着的这段内容的光标设置到最后去，true 折叠到 Range 的 start 节点，false 折叠到 end 节点。如果省略，则默认为 false .
  range.collapse(false);
  const sel = window.getSelection();
  sel!.removeAllRanges();
  sel!.addRange(range);
}

function mergeCells(){
  const {
    startRow,
    endRow,
    startCol,
    endCol,
  } =CalculateStartAndEnd (startCell.value,endCell.value);
  const rowSpan = endRow - startRow + 1;
  const colSpan = endCol - startCol + 1;

  for(let i = endRow;i >= startRow;i--){
    const tableRow = tiniTable.value.rows[i];
    for(let j = endCol; j >= startCol; j--){
      if(i == startRow && j == startCol){
        continue
      }
      const cellToHide = tableRow.cells[j];
      cellToHide.style.display = "none"
    }
  }  
  startCell.value.rowSpan = rowSpan;
  startCell.value.colSpan = colSpan;
  
}

onMounted(() => {
  window.addEventListener("click", cancelChosen);
});
onBeforeUnmount(() => {
  window.removeEventListener("click", cancelChosen);
});

export type CellType = {
  value: any;
  edit: boolean;
};
</script>

<template>
  <span>表格的行数：{{ props.rows }}</span>
  <span>表格的列数：{{ props.cols }}</span>
  <div class="menu">
    <span class="menu-item" @click ="mergeCells()">合并单元格</span>
    <span class="menu-item">新增一行</span>
    <span class="menu-item">新增一列</span>
    <span class="menu-item">删除行</span>
    <span class="menu-item">删除列</span>
  </div>
  <div>
    <table @click.stop ref="tiniTable">
      <tr v-for="row in tableInfo">
        <td
          @mousedown.stop="mousedown($event, data)"
          @mouseover.stop="mouseover($event)"
          @mouseup.stop="mouseup($event)"
          @dblclick="startEdit(data)"
          v-for="(data, index) in row"
          :index="index"
        >
          <div class="blur-textarea" v-if="!data.edit">
            {{ data.value }}
          </div>
         
          <div class="edit-textarea" v-if="data.edit" contenteditable="true">
            {{ data.value }}
          </div>
        </td>
      </tr>
    </table>
  </div>
</template>

<style scoped>
table {
  border-collapse: collapse;
  /* 合并单元格边框 */
  width: 500px !important;
  border: 1px solid red;
}

td {
  text-align: left;
  height: 40px;
  user-select: none;
  border: 1px solid black;
  max-width: 100px;
  min-width: 100px;
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
.edit-textarea {
  resize: none;
  min-height: 40px;
  max-height: 300px;
  word-wrap: break-word;
  /* overflow-x: hidden; */
  outline: 0;
}
.blur-textarea {
  resize: none;
  min-height: 40px;
  max-height: 300px;
  word-wrap: break-word;
  /* overflow-x: hidden; */
  outline: 0;
}
</style>
