<a name="FYDj3"></a>
#### 基础功能
- [x] 初始化单元格
- [x] 基础合并单元格
- [x] 单元格内容编辑
- [ ] 新增一行
- [ ] 新增一列
- [ ] 新增行考虑合并单元格
- [ ] 新增列考虑合并单元格
- [ ] 合并已合并单元格
- [ ] 删除行
- [ ] 删除列
- [ ] 删除合并单元格所在行和列
- [ ] 删除合并单元格对应行和列
- [ ] 单元格长宽可拖拉
<a name="A959Y"></a>
#### 性能优化

- [ ] 给部分操作加节流
- [x] 优化单元格编辑操作，使用普通html元素代替textarea实现编辑
- [x] 双击后光标自动锁定单元格末尾
- [ ] 表格宽高不能受合并单元格影响
<a name="hE9ga"></a>
#### 高级优化

- [ ] 抽取组件
- [ ] 独立表格渲染为render函数，与组件分离
- [ ] canvas替代js渲染
<a name="gbt4u"></a>
## 定义表格组件_TiniTableElement_
<a name="sBzP4"></a>
### 表格组件接收值

| rows | 表格的行数 |
| --- | --- |
| cols | 表格的列数 |

<a name="fxqUQ"></a>
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
<a name="KRbmd"></a>
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
<a name="DD3eo"></a>
## 实现表格选中效果
<a name="Ku7S2"></a>
### 定义以下属性
| isMouseDown | 记录鼠标是否按下 |
| --- | --- |
| startCell | 记录开始的单元格 |
| endCell | 记录结束的单元格 |

<a name="aHMYf"></a>
### 使用三个事件mousedown，mouseover，mouseup来实现单元格的选中
![](https://cdn.nlark.com/yuque/0/2023/jpeg/28358258/1685851494731-1df9b2de-1434-4cc0-bed5-ecb99770d511.jpeg)
<a name="f3C56"></a>
## 实现表格单元格编辑效果
<a name="ZtbDo"></a>
##### 现双击某个单元格，该单元格变为可点击状态。
<a name="ucegW"></a>
#### 方案1：采用textarea实现，textarea需要更改样式，实现起来复杂
![image.png](https://cdn.nlark.com/yuque/0/2023/png/28358258/1699192662718-ff9c51a4-ee6c-447a-ba49-fec80075e5ae.png#averageHue=%23fbfbfb&clientId=uc57901ac-381a-4&from=paste&height=343&id=u573508a6&originHeight=515&originWidth=905&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=5295&status=done&style=none&taskId=u04189177-244a-496d-8c3b-a4d7b77f0a2&title=&width=603.3333333333334)
<a name="vAo46"></a>
#### 方案2：采用div实现
**<div>** 元素是一个通用的块级容器，用于组织和布局其他 HTML 元素。当设置 **contenteditable="true"** 时，**<div>** 元素会变为可编辑状态，用户可以在其中输入和编辑文本，类似于一个简单的富文本编辑器。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/28358258/1699193020178-dcd6caf2-aa7e-4355-94ff-0bf27ce326d4.png#averageHue=%23f7f7f7&clientId=uc57901ac-381a-4&from=paste&height=219&id=ub9347377&originHeight=329&originWidth=648&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=6208&status=done&style=none&taskId=u25eed7a5-9073-4f9f-bc9b-526e8c01d6e&title=&width=432)
<a name="ncjgh"></a>
## 实现表格选中后光标定位到文字的最后
```vue
function set_focus(el: HTMLElement) {
  console.log(el);
  el.contentEditable = 'true'
  const range = document.createRange();
  range.selectNodeContents(el);
  range.collapse(false);
  const sel = window.getSelection();
  sel!.removeAllRanges();
  sel!.addRange(range);
}
```
<a name="TIKzG"></a>
## 实现表格合并
![image.png](https://cdn.nlark.com/yuque/0/2023/png/28358258/1699792244632-00f230d5-2931-4305-be50-ec51143f3cdd.png#averageHue=%23f8f2f2&clientId=u9a0f0037-a3f5-4&from=paste&height=249&id=ue6692216&originHeight=373&originWidth=691&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=4353&status=done&style=none&taskId=udf4ef40c-8fe0-4c3c-9338-184124dd23f&title=&width=460.6666666666667)<br />实现表格合并的关键是采用表格的rowSpan和colSpan属性。

1. 将开始的单元格的rowSpan和colSpan分别设置为选中单元格区域的跨行数和跨列数。
2. 隐藏其它的单元格
这里的开始的单元格总是最小索引值对应的那个单元格
