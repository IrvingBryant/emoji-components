# emoji-components
im中的表情~~~~~~
## 起手一张图后面然后听我给你编编编~~~~~~~~编花篮
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b4912078acd46eb9af3070619968d62~tplv-k3u1fbpfcp-watermark.image)
## 如何实现输入框带表情+文本+emoji
### 表情+文本+emoji实质是什么呢？？？
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1298e2c539744dda8e57a073d146a28e~tplv-k3u1fbpfcp-watermark.image)

**如上图注意啦敲烂黑板啦**

上面类似表情的展示的时候是一个img，但是他的实质内容是文本例如:smile:  :applaud: 这些可以把他当做是**占位符**，我们实际服务器存储的也是文本。

**以下文件为img表情占位符枚举**

```
{
  "表情": {
    ":smile:": "smile.png",
    ":applaud:": "applaud.png",
    ":cry:": "cry.png",
    ":laughing_out_loud:": "laughing_out_loud.png",
    ":wronged:": "wronged.png",
    ":embrace:": "embrace.png",
    ":shake_hands:": "shake_hands.png",
    ":thumbs_up:": "thumbs_up.png",
    ":love:": "love.png",
    ":rose_flower:": "rose_flower.png",
    ":anger:": "anger.png"
  }
}
```
## 表情面板如何渲染呢
### img表情渲染原理：
我猜各位大佬都已经猜到了使用雪碧图嘛。（不详细讲了！~~~~）  

**雪碧图**  
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/46bf2b026b0f484d81047bf92230d99b~tplv-k3u1fbpfcp-watermark.image)

**html模版**

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4d3ef7e01104ccaa6e17d46579f206b~tplv-k3u1fbpfcp-watermark.image)

**css-sprite**
```
[class*="sprite-"] {
  width:25px;
  height: 25px;
  background:url('./css_sprites.png') no-repeat;
  vertical-align: top;
  display: inline-block;
  margin: 5px;
  background-size:100px auto ;
}
.sprite-applaud {
  background-position: -0px -0px;
}
.sprite-embrace {
  background-position: -25px -0px;
}
.sprite-cry {
  background-position: -0px -25px !important;
}
.sprite-love {
  background-position: -25px -25px;
}
.sprite-smile {
  background-position: -50px  -0px!important;
}
.sprite-laughing_out_loud {
  background-position: -50px -25px;
}

.sprite-shake_hands {
  background-position: -0px -50px;
}
.sprite-wronged {
  background-position: -25px -50px;
}

.sprite-rose_flower {
  background-position: -50px -50px;
}
.sprite-thumbs_up {
  background-position: -75px -0;
}
```
### emoji表情渲染原理
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e76f005b0504ef18197e18af55212b6~tplv-k3u1fbpfcp-watermark.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cec88414cb58401d9b530d74a0c6509c~tplv-k3u1fbpfcp-watermark.image)

实质就是文本设置font-size既可以调整大小文本怎么渲染就怎么写就对了vue中就是{{}}

### 实现原理
实质表情就是文本（占位符），只是通过一系列的操作将占位符渲染成我们可见的表情（Img标签）如下代码，敲黑板重点来了**renderEmoji**方法匹配类似:smile:的占位符然后replace成Img 标签然后使用escHTML方法防XSS
```
/**
 * @export
 * @param {string} value
 * @returns {string}
 */

export function renderEmoji (value) {
  if (!value) return "";
  value = escHTML(value);
  Object.keys(emojiData).forEach( key => {
    value = value.replace(new RegExp(key, 'g'), createIcon(key))
  });
  return value
}
//防止XSS
function escHTML (str) {
  str = str + "";
  return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#39;");
}
function createIcon (item) {
  const value = emojiData[item]+"?max_age=31536000";
  return `<img src=${path}${value} style="vertical-align:middle" width="22px" height="22px" />`
}
```
- 聊天框渲染

	这个方法在聊天框内使用 v-html="renderEmoji(message.body.content)" 放心防Xss了

	![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56810a5ffeb14a1587256ad382a7d532~tplv-k3u1fbpfcp-watermark.image)

	这个方法的实质就是全局匹配占位符，然后将占位符replace成Img标签！~~

- 输入框渲染  

	输入框使用的**contenteditable="true"** 可编辑区域但是使用了contenteditable="true" 就没法使用v-html指令去渲染了，那我们要怎么做了呢？？  
    
   **思路**：就是我们直接往里面插标签不就OK了嘛！！~ 以下代码就是核心代码啦。既要**控制光标位置**、又要插入img表情节点以及文本节点
```
/**
 * cnt 是一个节点node（img、text节点）
 * 
*/
    cursorMove(cnt){
      var edit = this.$refs.userInput;
      const lastEditRange = this.lastEditRange;
      edit && edit.focus();
      var selection = getSelection();
      if (lastEditRange) {
        // 存在最后光标对象，选定对象清除所有光标并添加最后光标还原之前的状态
        selection.removeAllRanges();
        selection.addRange(lastEditRange);
      }
      // 判断选定对象范围是编辑框还是文本节点
      if (selection.anchorNode.nodeName != "#text") {
        // 创建新的光标对象
        let range = document.createRange();
        // 光标对象的范围界定为输入框节点
        range.selectNodeContents(edit);
        if (edit.childNodes.length > 0) {
          // 如果文本框的子元素大于0，则表示有其他元素，则按照位置插入表情节点
          for (var i = 0; i <= edit.childNodes.length; i++) {
            if (i == selection.anchorOffset) {
              edit.insertBefore(cnt, edit.childNodes[i]);
              // 光标位置定位在表情节点的最大长度
              range.setStart(edit, i+1);
            }
          }
        } else {
          // 否则直接插入一个表情元素
          edit.appendChild(cnt);
          // 光标位置定位在表情节点的最大长度
          range.setStart(edit, edit.childNodes.length);
        }
        // 使光标开始和光标结束重叠
        range.collapse(true);
        // 清除选定对象的所有光标对象
        selection.removeAllRanges();
        // 插入新的光标对象
        selection.addRange(range);
      } else {
        // 如果是文本节点则先获取光标对象
        let range = selection.getRangeAt(0);
        // 获取光标对象的范围界定对象，一般就是textNode对象
        var textNode = range.startContainer;
        // 获取光标位置
        var rangeStartOffset = range.startOffset;
        var textNodeContent = textNode.textContent
        //截取左文本、右文本
        var sliceLeftText = textNodeContent.slice(0,rangeStartOffset)
        var sliceRightText = textNodeContent.slice(rangeStartOffset)
        var textLeftNode = document.createTextNode(sliceLeftText);
        var textRightNode = document.createTextNode(sliceRightText)
        // 文本节点在光标位置处插入新的表情内容
        sliceLeftText && edit.insertBefore(textLeftNode, textNode);
        edit.insertBefore(cnt, textNode);
        sliceRightText && edit.insertBefore(textRightNode, textNode);
        //删除文本节点
        textNode.parentNode.removeChild(textNode);
        console.log('index',[].indexOf.call(cnt.parentNode.childNodes,cnt))
        range.setStart(cnt.parentNode,[].indexOf.call(cnt.parentNode.childNodes,cnt)+1);
        // 光标开始和光标结束重叠
        range.collapse(true);
        // 清除选定对象的所有光标对象
        selection.removeAllRanges();
        // 插入新的光标对象
        selection.addRange(range);
      }
      // 无论如何都要记录最后光标对象
      this.lastEditRange = selection.getRangeAt(0);
    }
```  
## 功能支持
### 文件拖拽发送功能  
可编辑区域监听一个drop事件就可以了啦！~~~~~~~~~
```
dropFIle(e) {
  const event = e || event 
  const df = event.dataTransfer;
  if (df.items) {
    df.items.forEach(async item => {
      if (item.kind == "file" && item.webkitGetAsEntry().isFile) {
        var file = item.getAsFile();
        if (file.type) {
          // TODO:文件传输对象此处写发送逻辑
          console.log('文件对象',file)
        }
      }
    });
  }
  event.preventDefault();
  event.stopPropagation();
},
```
### 文件复制发送功能
 目前支持文本和文件复制，当然后面可以支持img标签复制，这里不得不说contenteditable="true" 可编辑元素功能🐂🍺牛皮！~~~~~~  
```
//初始化监听粘贴事件  
mounted() {  
    this.$refs.userInput.addEventListener("paste", (e) => {
      this.handlePaste(e);
    });
  }

handlePaste(e) {
  const event = e || event 
  var items = event.clipboardData.items,
    len = items.length;
  for (var i = 0; i < len; i++) {
    var item = items[i];
    if (item.kind == "file") {
      var file = item.getAsFile();
      if (file) {
        // TODO:文件传输对象此处写发送逻辑
        console.log('复制文件到输入框触发',file)
      }
    }
    if (item.kind === "string" && item.type === "text/plain") {
      item.getAsString((str) => {
        var textNode = document.createTextNode(str);
        this.cursorMove(textNode)
      });
    }
  }
  event.preventDefault();
  event.stopPropagation();
}
```

### 表情发送功能
这操作就很简单了！~~~~点击表情面板选择表情然后输入框ENTER回车键就可以啦！~~~  
```
handleDownKey(event) {
  if (event.keyCode === 13 && !event.shiftKey) {
    // enter && not shift + enter
    this._submitText(event);
    event.preventDefault();
    return false;
  } else if (event.keyCode === 27) {
    // esc 键
    event.preventDefault();
  }
}
```

### 使用传送门全局所搜TODO:
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a07b09eec556468d9b17afdd4c4c73c4~tplv-k3u1fbpfcp-watermark.image)

## 开发过程中遇到的困难  
毫无疑问就是光标位置的控制，可真的是男上加男呀！~~~~~
- getSelection()
- selection.getRangeAt() 的 range 对象
- range.setStart()/range.setEnd()
- insertBefore 插入节点方法
[戳戳戳这里传送门！~~关于selection与Range的一些知识](https://zh.javascript.info/selection-range)

**注意：**  
	每次编辑、点击、选择输入框时都需要记录光标最终位置上面不累述了
    以及表情插入时每次动态计算光标位置等等！ 
 ```
  getSelectRange() {
    // 获取选定对象
    var selection = getSelection();
    // 设置最后光标对象
    this.lastEditRange = selection.getRangeAt(0);
  }
 ```
## [ 戳这里传送至git地址看代码吧觉得有用的小伙伴给个小星星吧！~~~~~](https://github.com/IrvingBryant/emoji-components)

## 严禁转载！~~~~
