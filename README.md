## 定义表格组件_TiniTableElement_
### 表格组件接收值

| rows | 表格的行数 |
| --- | --- |
| cols | 表格的列数 |

### 生成初始表格数据
```javascript
    /**
     * 初始化表格参数
     * @param rows 
     * @param cols 
     * @param content 
     */
    const TableInit = <T>(rows:number,cols:number,content:T):T[][]=> {
        const res = [];
        for(let i = 0; i < rows; i++){
            const temp = [];
            for(let j = 0; j < cols; j++){
                temp.push(content);
            }
            res.push([...temp])
        }
        return res;
    }
```
### 渲染表格
```html
<table>
  <tr v-for="row in tableInfo ">
    <td v-for="data in row">{{ data }}</td>
  </tr>
</table>
```
:::info
#### 效果
![image.png](https://cdn.nlark.com/yuque/0/2023/png/28358258/1685775193627-31b28b24-ef19-4703-9477-4b981104b237.png#averageHue=%23f9f8f8&clientId=u2ce50c84-59c3-4&from=paste&height=213&id=u9d39d788&originHeight=319&originWidth=941&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=3721&status=done&style=none&taskId=u0a7027ab-0835-4e83-8c49-f1f6a92dc14&title=&width=627.3333333333334)
:::
> 禁用文字选中效果

```vue
td {
    text-align: left;
    padding: 8px; 
    user-select:none;
    border: 1px solid black; /* 添加底部横线 */
  }
```
## 实现表格选中效果
### 定义以下属性
| isMouseDown | 记录鼠标是否按下 |
| --- | --- |
| startCell | 记录开始的单元格 |
| endCell | 记录结束的单元格 |

### 使用三个事件mousedown，mouseover，mouseup来实现单元格的选中
![](https://cdn.nlark.com/yuque/0/2023/jpeg/28358258/1685851494731-1df9b2de-1434-4cc0-bed5-ecb99770d511.jpeg)
```vue
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
    console.log("mouseup");
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
    for (let i = startRow; i <= endRow; i++) {
        let rows = tiniTable.value?.rows[i]
        for (let j = startCol; j <= endCol; j++) {
            let cell = rows.cells[j]
            cell.style.backgroundColor = 'gray';
            chosenCells.value.push(cell)
        }
    }
}
```
