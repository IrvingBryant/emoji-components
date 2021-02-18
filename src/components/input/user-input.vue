<template>
  <div>
    <Emoji @selectEmoji="_handleEmojiPicked"/>
    <form class="sc-user-input" :class="{ active: inputActive }">
      <!--用户输入框-->
      <div
        ref="userInput"
        role="button"
        tabindex="0"
        contenteditable="true"
        :placeholder="''"
        class="sc-user-input--text"
        @drop="dropFIle"
        @focus="setInputActive(true)"
        @blur="setInputActive(false)"
        @click="handleClick"
        @keyup.prevent="handleUpKey"
        @keydown="handleDownKey"
      ></div>
    </form>
  </div>
</template>

<script>
import { createImgNode } from "../emoji/emoji.js";
import Emoji from "../emoji/emoji.vue"
export default {
  data() {
    return {
      file: null,
      inputActive: false,
      fileTypeConf: {
        2: ["image/png", "image/jpg", "image/jpeg"],
        3: ["mp3", "amr", "wma", "flac", "aac", "m4a"],
        4: ["video/mp4", "video/avi", "video/mpeg4", "video/wma"]
      },
      lastEditRange: null
    };
  },
  components: {
    Emoji
  },
  mounted() {  
    this.$refs.userInput.addEventListener("paste", (e) => {
      this.handlePaste(e);
    });
  },
  methods: {
    getSelectRange() {
      // 获取选定对象
      var selection = getSelection();
      // 设置最后光标对象
      this.lastEditRange = selection.getRangeAt(0);
    },
    handleClick(event){
      const target = event.target
      const edit = this.$refs.userInput
      const selection = getSelection();
      if(target.nodeName === "IMG"){
        const range = document.createRange();
        let idx;
        // 光标对象的范围界定为输入框节点
        range.selectNodeContents(edit);
        //判断点击区域是否不是Img标签
        idx = [].indexOf.call(edit.childNodes,target) > -1 ? 
        [].indexOf.call(edit.childNodes,target) + 1 : 
        edit.childNodes.length
        range.setStart(edit, idx);
        // 使光标开始和光标结束重叠
        range.collapse(true);
        selection.removeAllRanges();
        // 插入新的光标对象
        selection.addRange(range);
      }
      this.lastEditRange = selection.getRangeAt(0);
    },
    setInputActive(onoff) {
      this.inputActive = onoff;
    },
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
    },
    //键盘抬起时获取最后光标范围
    handleUpKey() {
      //键盘抬起时获取最后光标范围
      this.getSelectRange()
    },
    focusUserInput() {
      this.$nextTick(() => {
        this.$refs.userInput.focus();
        const edit = this.$refs.userInput;
        const selection = getSelection();
        const range = selection.getRangeAt(0);
        if (edit.childNodes[0]) {
          range.setStart(edit.childNodes[0], edit.childNodes[0].length);
          range.setEnd(edit.childNodes[0], edit.childNodes[0].length);
          range.collapse(true);
          selection.removeAllRanges();
          selection.addRange(range);
        }
      });
    },
    _submitText() {
      let textChildNodes = this.$refs.userInput.childNodes
      let needSendText =''
      textChildNodes.forEach((v)=>{
        if(v.nodeName === 'IMG'){
          needSendText +=v.getAttribute('data-gemini-emoji')          
        }else if(v.nodeName === '#text'){
          needSendText +=v.textContent
        }
      })
      if (needSendText && needSendText.length > 0) {
        //TODO:调用发送接口
        console.log('待发送内容',needSendText)
        alert(needSendText)
      }
    },
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
    handlePaste(e) {
      const event = e || event 
      var items = event.clipboardData.items,
        len = items.length;
      var isHasTextHtml = Array.prototype.some.call(items,(v)=>{
        return v.type === 'text/html' && v.kind === "string"
      })
      for (var i = 0; i < len; i++) {
        var item = items[i];
        if (item.kind == "file") {
          var file = item.getAsFile();
          if (file) {
            // TODO:文件传输对象此处写发送逻辑
            console.log('复制文件到输入框触发',file)
          }
        }
        if (item.kind === "string" && item.type === "text/html" && isHasTextHtml) {
          let objE = document.createElement("div");
          item.getAsString((str) => {
            objE.innerHTML = str;
            let copyNodes = objE.childNodes
            copyNodes.forEach((v)=>{
              let emojiTxt = v.nodeName === 'IMG' && v.getAttribute('data-gemini-emoji')
              if(emojiTxt){
                var emojiText = createImgNode(emojiTxt)
                this.cursorMove(emojiText)         
              }
              if(v.nodeName !== 'IMG'){
                var textNode = document.createTextNode(v.innerText);
                this.cursorMove(textNode)
              }
            })
          });
        }
        if (item.kind === "string" && item.type === "text/plain" && !isHasTextHtml) {
          item.getAsString((str) => {
            var textNode = document.createTextNode(str);
            this.cursorMove(textNode)
          });
        }
      }
      event.preventDefault();
      event.stopPropagation();
    },
    _handleEmojiPicked(emoji,type='1') {
      let emojiText
      if(type==='1'){
        emojiText = createImgNode(emoji)
      }else{
        emojiText = document.createTextNode(emoji)
      }
      this.cursorMove(emojiText)
    },
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
  },
};
</script>

<style>
.sc-user-input {
  width: 100%;
  min-height: 55px;
  margin: 0px;
  position: relative;
  bottom: 0;
  display: flex;
  background-color: #f4f7f9;
  border-bottom-left-radius: 10px;
  border-bottom-right-radius: 10px;
  transition: background-color 0.2s ease, box-shadow 0.2s ease;
  text-align: left;
}
.sc-user-input-submit {
  border-radius: 4px;
  text-align: center;
  cursor: pointer;
}
.sc-user-input--text {
  width: 100%;
  height: 200px;
  resize: none;
  border: none;
  outline: none;
  border-bottom-left-radius: 10px;
  box-sizing: border-box;
  padding: 18px;
  font-size: 15px;
  font-weight: 400;
  line-height: 1.33;
  white-space: pre-wrap;
  word-wrap: break-word;
  color: #565867;
  -webkit-font-smoothing: antialiased;
  max-height: 200px;
  overflow: scroll;
  bottom: 0;
  overflow-x: hidden;
  overflow-y: auto;
  word-break: break-all;
}

.sc-user-input--text:empty:before {
  content: attr(placeholder);
  display: block; /* For Firefox */
  /* color: rgba(86, 88, 103, 0.3); */
  filter: contrast(15%);
  outline: none;
  cursor: text;
}

.sc-user-input--buttons {
  width: 150px;
  position: absolute;
  bottom: 30px;
  right: 30px;
  display: flex;
  justify-content: flex-end;
  align-items: center;
}

.sc-user-input--button:first-of-type {
  width: 40px;
}

.sc-user-input--button {
  width: 30px;
  margin-left: 2px;
  margin-right: 2px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.sc-user-input.active {
  box-shadow: none;
  background-color: white;
  box-shadow: 0px -5px 20px 0px rgba(150, 165, 190, 0.2);
}

.sc-user-input--button label {
  position: relative;
  height: 24px;
  padding-left: 3px;
  cursor: pointer;
}

.sc-user-input--button label:hover path {
  fill: rgba(86, 88, 103, 1);
}

.sc-user-input--button input {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  z-index: 99999;
  height: 100%;
  opacity: 0;
  cursor: pointer;
  overflow: hidden;
}

.file-container {
  background-color: #f4f7f9;
  border-top-left-radius: 10px;
  padding: 5px 20px;
  color: #565867;
}

.delete-file-message {
  font-style: normal;
  float: right;
  cursor: pointer;
  color: #c8cad0;
}

.delete-file-message:hover {
  color: #5d5e6d;
}

.icon-file-message {
  margin-right: 5px;
}
</style>
