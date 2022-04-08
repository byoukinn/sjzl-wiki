```js
/*
	script：DTS查询错误韩失败传输次数的流程v5
	author: 张家宝
	created: 2022年4月1日13:33:47
	edited:2022年4月6日15:44:46
	uses:  查询DTS每次运行记录的错误流程，丢到DTS数据库全选执行即可。
*/

var lastDays = 30 // 调整此处30（天） 可追溯至超过30天之前开始运行的记录，但运行时长会更久
var processVersionFlag = false
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
// 某些版本process节点不记录传输信息
if (db.P_DaaSProcessNodeInstance.find({pNodeType: 'process', faildTransCount: { $gt: 1 },}).length()) {
	processVersionFlag = true
}
// 查询DTS每次运行记录的错误流程
db.P_DaaSProcessNodeInstance.aggregate([
		{
			// 条件判断：只筛选了近一个月内的，想扩大范围，调整 ↓ 减日期数，减日期越大越往前追溯。
			$match:{
				pNodeType: processVersionFlag ? 'process' : { '$exists': true },
				endTime: { '$exists': true },
				faildTransCount: { $gt: 1 },
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
				'流程名称': '$Process.configName',
				'运行服务器': '$runServerId',
				'一级菜单': '$Category1.menuName', // good, very good
				'二级菜单': '$Category2.menuName',
				'运行开始时间': '$startTime',
				'运行结束时间': '$endTime',
				'失败传输次数': '$faildTransCount',
			}
		},
		{
			$sort: {
				'失败传输次数': -1
			}
		}
]);


```
