## 脚本介绍

该脚本可直接帮助现在直接快速生成DTS每天自动执行调度，每个每天执行调度间隔为15分钟，省去人工创建的时间。

##  使用方法

通过Robot或者Navicat连接到DTS的数据库（Mongodb）。
注：必须在RestCloud_DTS的数据库下面执行脚本，需要按步骤一步步执行，执行错误就需要自行删除相关数据重头执行。

## 执行步骤

### step1. 生成当前日期 

```js
/*
	script：将日期调度批量插入DTS平台v3
	author: 张家宝
	created: 2022年4月1日13:33:47
	edited:2022年4月6日15:44:46
	uses:  在RestCloud_DTS的数据库下面执行脚本。
*/

Date.prototype.format = function (fmt) {
  var o = {
    'M+': this.getMonth() + 1,                 // 月份
    'd+': this.getDate(),                    // 日
    'h+': this.getHours(),                   // 小时
    'm+': this.getMinutes(),                 // 分
    's+': this.getSeconds(),                 // 秒
    'q+': Math.floor((this.getMonth() + 3) / 3), // 季度
    'S': this.getMilliseconds()             // 毫秒
  }
  if (/(y+)/.test(fmt)) {
    fmt = fmt.replace(RegExp.$1, (this.getFullYear() + '').substr(4 - RegExp.$1.length))
  }
  for (var k in o) {
    if (new RegExp('(' + k + ')').test(fmt)) {
      fmt = fmt.replace(RegExp.$1, (RegExp.$1.length === 1) ? (o[k]) : (('00' + o[k]).substr(('' + o[k]).length)))
    }
  }
  return fmt
};
var createdCurTime = new Date();
createdCurTime = createdCurTime.format('yyyy-MM-dd hh:mm:ss');
```

### step2. 将调度的下级菜单插入到Mongodb数据库

```js
/*
	script：将日期调度批量插入DTS平台v3
	author: 张家宝
	created: 2022年4月1日13:33:47
	edited:2022年4月6日15:44:46
	uses:  在RestCloud_DTS的数据库下面执行脚本。
*/

db.P_AppMenuItemConfig.insertMany([
  {
    "_id": ObjectId("5d9a8b1d3096666670e67a00"),
    "categoryId" : "daas.scheduler",
    "parentNodeId" : "home",
    "nodeId" : "dayrule_div",
    "menuName" : "每天固定调度合集",
    "sortNum" : "1001",
    "leafFlag" : false,
    "appId" : "daas",
    "createTime" : createdCurTime,
    "creator" : "system",
    "creatorName" : "系统设置",
    "editTime" : createdCurTime,
    "editor" : "system",
    "editorName" : "系统设置",
    "id" : "5b7a8b1d3096666670e67a01"
  },

	{
    "_id": ObjectId("5d9a8b1d3096666670e67a01"),
    "categoryId" : "daas.scheduler",
    "parentNodeId" : "dayrule_div",
    "nodeId" : "dayrule_0_3",
    "menuName" : "每天凌晨0点到3点",
    "sortNum" : "1001",
    "leafFlag" : false,
    "appId" : "daas",
    "createTime" : createdCurTime,
    "creator" : "system",
    "creatorName" : "系统设置",
    "editTime" : createdCurTime,
    "editor" : "system",
    "editorName" : "系统设置",
    "id" : "5b7a8b1d3096666670e67a01"
	},

  {
    "_id": ObjectId("5d9a8b1d3096666670e67a02"),
    "categoryId" : "daas.scheduler",
    "parentNodeId" : "dayrule_div",
    "nodeId" : "dayrule_3_6",
    "menuName" : "每天凌晨3点到6点",
    "sortNum" : "1001",
    "leafFlag" : false,
    "appId" : "daas",
    "createTime" : createdCurTime,
    "creator" : "system",
    "creatorName" : "系统设置",
    "editTime" : createdCurTime,
    "editor" : "system",
    "editorName" : "系统设置",
    "id" : "5b7a8b1d3096666670e67a02"
  },

  {
    "_id": ObjectId("5d9a8b1d3096666670e67a03"),
    "categoryId" : "daas.scheduler",
    "parentNodeId" : "dayrule_div",
    "nodeId" : "dayrule_6_9",
    "menuName" : "每天早上6点到9点",
    "sortNum" : "1001",
    "leafFlag" : false,
    "appId" : "daas",
    "createTime" : createdCurTime,
    "creator" : "system",
    "creatorName" : "系统设置",
    "editTime" : createdCurTime,
    "editor" : "system",
    "editorName" : "系统设置",
    "id" : "5b7a8b1d3096666670e67a03"
  },

  {
    "_id": ObjectId("5d9a8b1d3096666670e67a04"),
    "categoryId" : "daas.scheduler",
    "parentNodeId" : "dayrule_div",
    "nodeId" : "dayrule_9_12",
    "menuName" : "每天早上9点到12点",
    "sortNum" : "1001",
    "leafFlag" : false,
    "appId" : "daas",
    "createTime" : createdCurTime,
    "creator" : "system",
    "creatorName" : "系统设置",
    "editTime" : createdCurTime,
    "editor" : "system",
    "editorName" : "系统设置",
    "id" : "5b7a8b1d3096666670e67a04"
  },

  {
    "_id": ObjectId("5d9a8b1d3096666670e67a05"),
    "categoryId" : "daas.scheduler",
    "parentNodeId" : "dayrule_div",
    "nodeId" : "dayrule_12_15",
    "menuName" : "每天中午12点到15点",
    "sortNum" : "1001",
    "leafFlag" : false,
    "appId" : "daas",
    "createTime" : createdCurTime,
    "creator" : "system",
    "creatorName" : "系统设置",
    "editTime" : createdCurTime,
    "editor" : "system",
    "editorName" : "系统设置",
    "id" : "5b7a8b1d3096666670e67a05"
  },

  {
    "_id": ObjectId("5d9a8b1d3096666670e67a06"),
    "categoryId" : "daas.scheduler",
    "parentNodeId" : "dayrule_div",
    "nodeId" : "dayrule_15_18",
    "menuName" : "每天下午15点到18点",
    "sortNum" : "1001",
    "leafFlag" : false,
    "appId" : "daas",
    "createTime" : createdCurTime,
    "creator" : "system",
    "creatorName" : "系统设置",
    "editTime" : createdCurTime,
    "editor" : "system",
    "editorName" : "系统设置",
    "id" : "5b7a8b1d3096666670e67a06"
  },

  {
    "_id": ObjectId("5d9a8b1d3096666670e67a07"),
    "categoryId" : "daas.scheduler",
    "parentNodeId" : "dayrule_div",
    "nodeId" : "dayrule_18_21",
    "menuName" : "每天晚上18点到21点",
    "sortNum" : "1001",
    "leafFlag" : false,
    "appId" : "daas",
    "createTime" : createdCurTime,
    "creator" : "system",
    "creatorName" : "系统设置",
    "editTime" : createdCurTime,
    "editor" : "system",
    "editorName" : "系统设置",
    "id" : "5b7a8b1d3096666670e67a07"
  },

  {
    "_id": ObjectId("5d9a8b1d3096666670e67a08"),
    "categoryId" : "daas.scheduler",
    "parentNodeId" : "dayrule_div",
    "nodeId" : "dayrule_21_24",
    "menuName" : "每天晚上21点到24点(不含24)",
    "sortNum" : "1001",
    "leafFlag" : false,
    "appId" : "daas",
    "createTime" : createdCurTime,
    "creator" : "system",
    "creatorName" : "系统设置",
    "editTime" : createdCurTime,
    "editor" : "system",
    "editorName" : "系统设置",
    "id" : "5b7a8b1d3096666670e67a08"
  },

]);
```

## step3. 将时间调度插入到Mongodb数据库

```js
/*
	script：将日期调度批量插入DTS平台v3
	author: 张家宝
	created: 2022年4月1日13:33:47
	edited:2022年4月6日15:44:46
	uses:  在RestCloud_DTS的数据库下面执行脚本。
*/

// 与clockNames的第二层中括号一一对应(下标对照)
var clockCatas = ['凌晨','早上','中午','下午','晚上',];
var clockNames = [
   ['0','1','2','3','4','5'],
   ['6','7','8','9','10','11'],
   ['12','13'],
   ['14','15','16','17'],
   ['18','19','20','21','22','23'],
];
// 与minuteCrons一一对应
var minuteNames = ['0', '15', '30', '45'];
var minuteCrons  = [0, 15, 30, 45];

// 运行时会根据clockCount除以3获得下标一一对应(下标对照)
var categoryIds = [
  'dayrule_0_3', 
  'dayrule_3_6',
  'dayrule_6_9',
  'dayrule_9_12',
  'dayrule_12_15',
  'dayrule_15_18',
  'dayrule_18_21',
  'dayrule_21_24',
];
var clockCount = 0;
var idCount = 1; // 不允许mongo文档插入时重复的id主键
var tempTable = [];
clockCatas.forEach((cata, i) => {
    clockNames[i].forEach(clock => {
        minuteNames.forEach((min, j) => {
            // 拼接时分秒变成
            timeName = `每天${cata}${clock}时${min}分`;
            var timeCron = `0 ${minuteCrons[j]} ${clockCount} * * ?`;
            tempTable.push({
              ruleName: timeName,
              expression: timeCron,
              categoryId: categoryIds[Math.floor(clockCount/3)]
            });
            // 文档插入数据库
            var idStr = idCount.toString().padStart(3, '0');
            var id = "5d4a8b1d2098656671a7b" + idStr;
            db.P_DaaSSchedulerRuleConfig.insertOne({
              "_id" : ObjectId(id),
              "ruleId" : "DAAS_TIME_150011"+idStr,
              "ruleName" : timeName,
              "expression" : timeCron,
              "categoryId" : categoryIds[Math.floor(clockCount/3)],
              "state" : "1",
              "id" : id,
              "appId" : "daas",
              "createTime" : createdCurTime,
              "creator" : "system",
              "creatorName" : "系统设置",
              "editTime" : createdCurTime,
              "editor" : "system",
              "editorName" : "系统设置"
            });
            idCount += 1;
        })
        clockCount += 1;
    }) 
});
```
