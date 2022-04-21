## 脚本介绍

该脚本通过查询DTS每次运行超过15分钟的流程运行记录，来帮助判断哪些流程运行超时异常，便于整改流程。

##  使用方法

连接至DTS数据库mongodb，要求用户空间为`Restcloud_DTS`，将以下内容全选复制直接执行即可观察结果。

## 各项参数

### timeoutMins 

调整该项，仅查询运行记录超过`timeoutMins`（分钟）的流程

### lastDays

调整该项，仅查询`lastDays`（天）内的流程运行记录


## 脚本内容
```js
/*
	script：DTS查询错误含失败传输次数的流程
	author: 张家宝
	created: 2022年4月1日13:33:47
	uses:  查询DTS每次运行记录超过15分钟的（可调整timeoutMins），丢到DTS数据库全选执行即可。

*/
var timeoutMins = 15 // 调整此处 15（分钟） 延长数据交换超时范围
var lastDays = 30 // 调整此处30（天） 可追溯至超过30天之前开始运行的记录，但运行时长会更久

// 日期格式化
Date.prototype.format = function(fmt)   
{ //author: meizz   
  var o = {   
    "M+" : this.getMonth()+1,                 //月份   
    "d+" : this.getDate(),                    //日   
    "h+" : this.getHours(),                   //小时   
    "m+" : this.getMinutes(),                 //分   
    "s+" : this.getSeconds(),                 //秒   
    "q+" : Math.floor((this.getMonth()+3)/3), //季度   
    "S"  : this.getMilliseconds()             //毫秒   
  };   
  if(/(y+)/.test(fmt))   
    fmt=fmt.replace(RegExp.$1, (this.getFullYear()+"").substr(4 - RegExp.$1.length));   
  for(var k in o)   
    if(new RegExp("("+ k +")").test(fmt))   
  fmt = fmt.replace(RegExp.$1, (RegExp.$1.length==1) ? (o[k]) : (("00"+ o[k]).substr((""+ o[k]).length)));   
  return fmt;   
}

// 查询DTS每次运行记录的错误流程
db.P_DaaSProcessNodeInstance.aggregate([
		{
			// 条件判断：只筛选了近一个月内的，想扩大范围，调整 ↓ 减日期数，减日期越大越往前追溯。
			$match:{
				pNodeId: 'process',
//				faildTransCount: { $gt: 1 },
				startTime: { $gte: (new Date(ISODate().getTime() - 1000*60*60*24*lastDays)).format('yyyy-MM-dd hh:mm:ss') },
			}
		},
		{
			$lookup: { // 左连接
				from: 'P_DaaSProcessModelConfig', // 关联到外联表
				localField: 'processId', // 本表关联的字段
				foreignField: 'id', // 外联表关联的字段
				as: 'Process'
			},
		}, 
   		{
			$unwind: { 
				path: '$Process',
				preserveNullAndEmptyArrays: true // 空的数组也拆分
			}
		},
		{
			$lookup: { // 左连接
				from: 'P_AppMenuItemConfig', // 关联到外联表
				localField: 'Process.categoryId', // 本表关联的字段
				foreignField: 'nodeId', // 外联表关联的字段
				as: 'Category2'
			}
		},
		{
			$unwind: { 
				path: '$Category2',
				preserveNullAndEmptyArrays: true // 空的数组也拆分
			}
		},
		{
			$lookup: { // 左连接
				from: 'P_AppMenuItemConfig', // 关联到外联表
				localField: 'Category2.parentNodeId', // 本表关联的字段
				foreignField: 'nodeId', // 外联表关联的字段
				as: 'Category1'
			}
		},	
		{
			$unwind: { // 拆分子数组
				path: '$Process',
				preserveNullAndEmptyArrays: true // 空的数组也拆分
			}
		},
		{
			$unwind: { // 拆分子数组
				path: '$Category1',
				preserveNullAndEmptyArrays: true // 空的数组也拆分
			}
		},
		{ // 映射，去掉这部分可以看到全部信息
			$project: {
				_id: 0,
				'流程名称': '$pNodeName',
				'运行服务器': '$runServerId',
				'一级菜单': '$Category1.menuName',
				'二级菜单': '$Category2.menuName',
				'运行开始时间': '$startTime',
				'运行结束时间': '$endTime',
				'失败传输次数': '$faildTransCount',
				'间隔分钟': {$divide: [{$subtract:[{'$dateFromString': {dateString: '$endTime'} }, {'$dateFromString': {dateString: '$startTime'}}]}, 60*1000]},
			}
		},
		{
			$match: {
				'间隔分钟': {$gte: timeoutMins} 
			}
		},
    	{
            $sort: {
                '间隔分钟': -1,
            }
        },
]);

```