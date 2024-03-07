<script setup>
import { parseOfdDocument, renderOfd } from 'ofd.js'
import { ref } from 'vue'

let screenWidth = document.body.clientWidth - 88
// let file = require('../assets/111.ofd')
// console.log("file", file);
// getOfdDocumentObj(file, screenWidth)
function getOfdDocumentObj(file, screenWidth) {
    console.log("get", file, screenWidth);
  parseOfdDocument({
    ofd: file,
    success(res) {
        console.log("res", res);
      const ofdObj = res[0]
      // 渲染OFD
      const divs = renderOfd(screenWidth, ofdObj)
      // 显示OFD渲染的内容
      displayOfdDiv(divs)
      analyseTexts()
      //
      highlight(['青团'])
    },
    fail(error) {
      console.log('error', error)
    }
  })
}

function displayOfdDiv(divs) {
    console.log("divs", divs);
  let contentDiv = document.getElementById('content')
  contentDiv.innerHTML = ''
  for (const div of divs) {
    contentDiv.appendChild(div)
  }
}

let textElements = []
let textNodes = []
let content = []
let highlights = []
// 初始化文本
function analyseTexts() {
  textElements = Array.from(document.getElementsByTagName('text'))
  let length = 0
  textNodes = textElements.map((element) => {
    let startIdx = length,
      endIdx = length + element.innerHTML.length
    length = endIdx
    return {
      text: element.innerHTML,
      startIdx,
      endIdx
    }
  })
  content = textNodes.map(({ text }) => text).join('')
}
// 高亮
function highlight(arr) {
  CSS.highlights.clear()
  highlights = []
  arr.forEach((str) => {
    const res = getMatchList(content, str)
    highlightMatchList(textElements, textNodes, res)
  })
}
// 获取正则匹配结果
function getMatchList(content, keyword) {
  const characters = [...'[]()?.+*^${}:'].reduce((r, c) => ((r[c] = true), r), {})
  console.log(characters);
  keyword = keyword
    .split('')
    .map((s) => (characters[s] ? `\\${s}` : s))
    .join('[\\s\\n]*')
  console.log('keyword', keyword)
  const reg = new RegExp(keyword, 'gmi')
  return [...content.matchAll(reg)]
}
// 高亮匹配的结果
function highlightMatchList(textElements, textNodes, matchList) {
  for (let i = matchList.length - 1; i >= 0; i--) {
    const match = matchList[i]
    const matchStart = match.index,
      matchEnd = matchStart + match[0].length
    for (let index = 0; index < textNodes.length; index++) {
      //
      const { startIdx, endIdx } = textNodes[index] // 文本内容、文本在拼接串中开始、结束索引
      if (endIdx <= matchStart) {
        continue
      }
      if (startIdx >= matchEnd) {
        break
      }
      let element = textElements[index]
      const nodeMatchStartIdx = Math.max(0, matchStart - startIdx)
      const nodeMatchLength = Math.min(endIdx, matchEnd) - startIdx - nodeMatchStartIdx
      const range = new Range()
      range.setStart(element.firstChild, nodeMatchStartIdx)
      range.setEnd(element.firstChild, nodeMatchStartIdx + nodeMatchLength)
      highlights.push(range)
    }
    // 创建高亮对象
    const searchResultsHighlight = new Highlight(...highlights.flat())
    // 注册高亮
    CSS.highlights.set('search-results', searchResultsHighlight)
  }
}
let file = ''
let fileRef = ref(null)
function uploadFile() {
  file = null
}
function fileChanged(e) {
    console.log("file", e);
  file = e.target.files[0]
  console.log("file", file);
  getOfdDocumentObj(file, screenWidth)
//   fileRef.value.file.value = null
}
</script>
<template>
  <div>
    <div class="upload-icon" @click="uploadFile">
      <div class="upload-icon">打开OFD</div>
      <font-awesome-icon icon="cloud-upload-alt" />
      <input type="file" ref="fileRef" class="hidden" accept=".ofd" @change="fileChanged" />
    </div>
  </div>
  <div class="box">
    <div id="content"></div>
  </div>
</template>
<style scoped>
::highlight(search-results) {
  background-color: #f06;
  color: white;
}
.box {
    overflow: scroll;
}
</style>
