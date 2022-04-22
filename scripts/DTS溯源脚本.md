## 脚本介绍

该脚本适用于DTS的溯源。

##  使用方法

1. 按`F12`打开浏览器的开发者控制台
2. 将压缩后的代码粘贴进去敲回车
3. 打开dts的源表节点，出现弹出框后，按alt+x（或者手动输入`auto()`)
4. 网速差的耐心等待跳转到字段页面，注意，如果打印origin是空，则尝试alt+p增加延迟时间
5. alt+q可以让折起来的面板展开，展开的面板折起

## 更新历史

版本从最新到最旧

| 版本号 | 更新内容                                                     |
| ------ | ------------------------------------------------------------ |
| v9     | DTS更新组件，改名，修复bug，增加auto()时的提示               |
| v8     | sql分离器的解析力增加，解决结果表的最后一行无法显示的问题，结果表的精准度增加 |
| v7     | 修复展开和折叠不一定成功的bug，无论如何都将结果打印          |
| v6     | 增加了时长可控制，按alt+p调出控制框，按alt+q切换血缘关系折叠状态，判断字段是否正确的方式改成为由字段页面取文字来对比，增加case when的解析力，兼容性更强 |
| v5     | 增加alt+x快捷键启动脚本                                      |
| v4     | 增加便于识别                                                 |
| v3     | 让全自动展开后，自动关闭，并且再次展开错误会变蓝色           |
| v2     | 增加系统判定错误与成功的颜色的功能                           |
| v1     | 全自动展开，让系统自动判断                                   |

## 脚本内容 

```js
/**
 * scriptName：dts溯源脚本
 * author：张家宝
 * lastEdit：2021-3-2 16:30:44
 * feature：
  1.全自动点击溯源，打开来源节点，控制台输入auto()就可以实现
  2.从sql分析排查哪些字段有别名
  3.在溯源界面，无别名或者别名相同的情况，标绿色，需要修改的部分标红色，修改后的部分标蓝色
  4.增加全量版本的sql解析器，不通过\n符来拆除行，增加了对多行case when的兼容
  5.增加快捷键，打开来源节点页面，按alt+x就自动执行脚本
  6.增加了时长可控制，
    按alt+p调出控制框，
    按alt+q切换血缘关系折叠状态，判断字段是否正确的方式改成为由字段页面取文字来对比，增加case when的解析力，兼容性更强
    按alt+s查看sql
  7.修复展开和折叠不一定成功的bug，无论如何都将结果打印
  8.sql分离器的解析力增加，解决结果表的最后一行无法显示的问题，结果表的精准度增加
 */
// 嵌入样式
var $ = sel => document.querySelectorAll(sel)
var preloadMilsec = 1200
var rowExpended = false
var origin = ''
function auto() {
  console.log('开始自动执行懒人脚本，请等待 ' + preloadMilsec + ' 秒')
  var l = $('.ant-modal-body .ant-tabs-tab[aria-selected="true"]')[0].parentElement.childNodes
  l[1].click(); // SQL界面
  var iframe = $('.ant-modal-body iframe')[0]
  setTimeout(function () {
    var sql = iframe.contentWindow.getCode()
    origin = sql
    var mismatchColDatas = getMismatchAndPrintRelation(sql)
    setTimeout(function() {
      l[2].click(); // 字段界面
      $('.ant-tabs-tabpane-active .ant-table-row-collapsed').forEach(e => e.click());
      $('.ant-tabs-tabpane-active .ant-tabs-tab[aria-selected=false]').forEach(e => e.click()); // 切换到血缘关系
      $('.ant-table-placeholder').forEach(e => { 
        var panel = e.parentElement.parentElement.parentElement.parentElement.parentElement.parentElement
        panel.querySelectorAll('.ant-btn-primary')[0].click() // 找到没有血缘关系的，自动添加
        var rowKey = panel.parentElement.parentElement.parentElement.parentElement.parentElement.parentElement.parentElement.dataset.rowKey.split('-')[0]
        // 给自动添加的上色，错则marked，正确则normal
        var rowMain = $('.ant-tabs-tabpane-active .ant-table-wrapper .ant-table-tbody>.ant-table-row[data-row-key="'+rowKey+'"]')[0]
        // var condText = rowMain.querySelector('.ant-table-row-cell-break-word').innerText
        // var found = mismatchColDatas.find(e => e['目标'] == condText)
        // var colorType = (found && found['源表'] != condText) || mismatchColDatas.length < 1 ? 'marked-row' : 'normal-row'
        rowMain.classList.add('marked-row') 
        $('.ant-tabs-tabpane-active .ant-table-row[data-row-key="'+rowKey+'"] .ant-table-row-expand-icon')[0].addEventListener('click', function() {
          rowMain.classList.add('modified-row')}
        )
        // 
      })
 
      
      // $('.ant-tabs-tabpane-active .ant-table-row-expand-icon-cell').forEach((e, i) => {
      //  // console.log(i, e)
      //  mismatchColDatas.includes(i) 
      //      ? e.classList.add('marked-row')
      //      : e.classList.add('normal-row')
      //  e.querySelector('.ant-table-row-expand-icon').addEventListener('click', function() {e.classList.add('modified-row')})
      // })
      }, 1000)
  }, preloadMilsec)
}


function printSql() {
  console.log('开始自动执行printSql脚本，请等待 ' + preloadMilsec + ' 秒')
  var l = $('.ant-modal-body .ant-tabs-tab[aria-selected="true"]')[0].parentElement.childNodes
  l[1].click(); // SQL界面
  var iframe = $('.ant-modal-body iframe')[0]
  setTimeout(function () {
    var sql = iframe.contentWindow.getCode()
    origin = sql
    var mismatchColDatas = getMismatchAndPrintRelation(sql)
  }, preloadMilsec)
}


function getMismatchAndPrintRelation( sql ) {
  // TODO：distinct 的时候
  var a_FuncReg   = /\((\w*\.?\w+),.*[ASas]{2}\s+(\w+)/ // 有函数 跟逗号 写as
  var n_FuncReg   = /\((\w*\.?\w+),.*\s+(\w+)/ // 有函数  跟逗号 没写as
  var a_FuncReg1  = /\((\w*\.?\w+),?.*[ASas]{2}\s+(\w+)/ // 有函数 写as
  var n_FuncReg1  = /\((\w*\.?\w+),?.*\s+(\w+)/ // 有函数 没写as
  var a_DstReg  = /distinct\s*(\w*\.?\w+).*[ASas]{2}\s+(\w+)/ // distinct的 写as
  var n_DstReg  = /distinct\s*(\w*\.?\w+).*\s+(\w+)/ // distinct的 写as
  var a_CaseReg   = /case\s*(\w*\.?\w+)\s*when(\s|.)*end(\s|.)*[asAS]{2}\s*(\w*\.?\w+)/ // case when的
  var n_CaseReg   = /case\s*(\w*\.?\w+)\s*when(\s|.)*end(\s|.)*(\w*\.?\w+)/ // case when的
  var a_CaseReg1  = /case\s*when\s*(\w*\.?\w+)(\s|.)*end(\s|.)*[ASas]{2}\s+(\w+)/ // case when的 写as
  var n_CaseReg1  = /case\s*when\s*(\w*\.?\w+)(\s|.)*end(\s|.)*(\w+)/ // case when的 没写as
  var a_QuotaReg  = /(\'?\w*\.?\w+\'?).*[ASas]{2}\s+(\w+)/ // 前面是字符串 写as
  var n_QuotaReg  = /(\'?\w*\.?\w+\'?).*\s+(\w+)/ // 前面是字符串 没写as
  var regs = [
    a_FuncReg ,
    n_FuncReg ,
    a_FuncReg1,
    n_FuncReg1,
    a_DstReg  ,
    n_DstReg  ,
    a_CaseReg ,
    n_CaseReg ,
    a_CaseReg1,
    n_CaseReg1,
    a_QuotaReg,
    n_QuotaReg,
  ]
  // 数据分组处理
  let arr = splitSql(sql)
  console.log(arr)
  let colNums = []
  // 显示
  let datas = []
  let found = false
  let err = false
  for (var k = 0; k < arr.length; k++) {
    for (var i = 0; i < regs.length; i++) {
      var s = arr[k].match(regs[i])
      if (s && s.length) {
        if (s[1] != s[s.length - 1].toLowerCase() && s[1].indexOf("'")) {
          console.log('reg:', k, i, regs[i])
          found = true
          datas.push({'源表': s[1], '目标': s[s.length - 1]})
          colNums.push(k)

          if (['case', 'then', 'else'].includes(s[1])) {
            err = true
            break
          }
        }
        break
      }
    }
  }
  if (err) {
    console.group('%cERRO%c该SQL过于复杂，所列表不能作为参考', 'padding:2px 4px;background: pink', 'padding:2px 4px;background: #666;color:white')
    console.log(origin)
      console.groupCollapsed('%c参考表（仅供参考，点击查看）', 'padding:2px 4px;background: pink')
      console.table(datas, ['源表', '目标'])
      console.groupEnd()
    console.groupEnd()
  } else if (found) {
    console.table(datas, ['源表', '目标'])
    console.groupCollapsed('%c参考表（仅供参考，点击查看）', 'padding:2px 4px;background: #666')
    console.log(origin)
    console.groupEnd()

  } else {
    // NO AS OR STAR
    console.group('%cNAOS%c没有可对比项或原sql是星号', 'padding:2px 4px;background: greenyellow', 'padding:2px 4px;background: #666;color:white')
    console.log(origin)
    console.groupEnd()
  }
  return datas
}

function splitSql(originSql) {
  var lowerCaseSql = originSql.toLowerCase()
  sql = lowerCaseSql.substr(7, lowerCaseSql.indexOf('from')-7).trim()
  var stack = []
  var ret = []
  var caseIndexs = [] // 存有case的所有下标
  var endIndexs = [] // 存有end的所有下标
  var start = 0, cnt = 0
  var prevChar = ''
  var nextChar = ''
  var isStr = false
  var inCase = false

  while(cnt != -1) {
      cnt = sql.indexOf('case', cnt + 1)
      caseIndexs.push(cnt)
      if (cnt == -1) break;
      cnt = sql.indexOf('end', cnt + 1)
      endIndexs.push(cnt)
  }
  caseIndexs.pop() // 最后一个一定是 -1， 不需要

  for (var i = 0, len = sql.length; i < len; i++) {
    var c = sql[i]
    // 找case when end 
    if (i == caseIndexs[0]) {
      var k = sql.indexOf(',', endIndexs[0]) + 1
      ret.push(sql.substr(i, k-i))
      caseIndexs.shift()
      endIndexs.shift()
      i = k
      start = k
    }
      var prevChar = i > 0 ? sql[i-2] : '' 
    var nextChar = i != sql.length ? sql[i] : '' 

      switch (c) {
          case '(': stack.push(1); break;
          case ')': stack.pop(); break;
          // 有注释的地方清洗一下
          case "'": isStr = !isStr; break; // 字符串里的不算
          case '-': if (nextChar == '-' && !isStr && !stack.includes(2)) {stack.push(2)}; break;
          case '/': prevChar == '*' ? (stack.pop(), start=i) : nextChar == '*' ? stack.push(2) : undefined; break;
          case '\n': if (stack[stack.length-1] == 2) {stack.pop(); start=i+1}; break;
          case ',': if (!stack.length) { 
              ret.push(sql.substr(start, i-start).trim()); 
              start = i+1;
          }; break;
      }
  }
  var r = sql.substr(start, sql.length)
  var offset = r.indexOf('--') != -1 ? r.indexOf('--') - 1 : r.length
  ret.push(r.substr(0, offset).trim()); 
  return ret
}

function configurePreloadTime() {
  let t = prompt('输入需要的等待延迟时间（默认200ms）（网络比较卡的调高点）', 1200)
  if (isNaN(t)) {
    while (!isNaN(t)) {
      alert('请输入一个数字（如果要1秒则输入1000）')
      t = prompt('输入需要的等待延迟时间（默认1200ms）（网络比较卡的调高点）', 1200)
    }
  }
  preloadMilsec=parseInt(t)
}

function toggleAllRowExpended() {
  var es = $('.ant-tabs-tabpane-active .ant-table-row-collapsed')
  if (es.length) {
    es.forEach(e => e.click());
  } else {
    $('.ant-tabs-tabpane-active .ant-table-row-expanded').forEach(e => e.click());
  }
}

function createStyle() {
  // 创建样式
  var styleNode = document.createElement('style')
  styleNode.appendChild(document.createTextNode('.marked-row{background: pink}.normal-row{background: lightgreen}.modified-row{background: lightblue}'))
  document.head.appendChild(styleNode)
}

function setupAllStepAndConfiguration() {
  createStyle()
  window.addEventListener("keyup", e => {
      if(e.code == 'KeyX' && e.altKey) {auto()} // 开始执行脚本
      else if(e.code == 'KeyS' && e.altKey) {printSql()} // 查看sql
      else if(e.code == 'KeyP' && e.altKey) {configurePreloadTime()} // 设置预加载时间
      else if(e.code == 'KeyQ' && e.altKey) {toggleAllRowExpended()} // 翻盖
  })
}

setupAllStepAndConfiguration()

// 压缩后的代码
```
