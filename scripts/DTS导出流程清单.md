## 脚本介绍

该脚本适用于DTS的溯源。

##  使用方法

1. 试用Navicat打开DTS的mongodb数据库（目前仅测试navicat成功）
2. 

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
db.getCollection("P_AppMenuItemConfig").update({
	_id: ObjectId("9900ff01000020002224222f")
},
{
	_id: ObjectId("9900ff01000020002224222f"),
	categoryId: "daas.routerflow",
	parentNodeId: "home",
	nodeId: "DAAS_MENU_789000001",
	menuName: "原所有流程内含",
	sortNum: "1001",
	openType: "1",
	leafFlag: false,
	count: NumberInt("0"),
	appId: "daas",
	createTime: "2022-01-01 00:00:00",
	creator: "admin",
	creatorName: "管理员",
	editTime: "2022-01-01 00:00:00",
	editor: "admin",
	editorName: "管理员",
	id: "9900ff01000020002224222f"
},
{
	upsert: true
}); // 有则更新，没有则创建
// 移动文件夹
db.getCollection('P_DaaSProcessModelConfig').updateMany({ categoryId: 'all' },{
	$set: {
		categoryId: 'DAAS_MENU_789000001'
	}
})

if (db.getCollection('P_DaaSProcessModelConfig').find({
	categoryId: 'DAAS_MENU_789000001'
}).length() == 0) {
	db.getCollection('P_AppMenuItemConfig').deleteOne({
		_id: ObjectId("9900ff01000020002224222f")
	})
}

// 查询DTS的模型/流程清单
var processLists = db.P_DaaSProcessModelConfig.aggregate([
// {
// 	// 条件判断：只筛选状态为自动的流程
// 	$match:{
//		state: 1, 
// 	}
// },
// {
// 	$limit: 35 // 调试时使用
// },

{
	$lookup: { // 左连接
		from: 'P_DaaSProcessNodeConfig',
		let: {
			processId: '$id'
		},
		pipeline: [{
			$match: {
				pNodeType: 'router',
				sourceId: 'E00001',
				// 一定要保证的线的起始节点是该ID
				$expr: {
					$eq: ["$processId", "$$processId"]
				},
			}
		},
		{
			$sort: {
				pNodeId: -1,
			}
		},
		{
			$limit: 1
		},
		{
			$project: {
				targetId: 1,
			},
		}],
		as: 'Nodes'
	}
},
{
	$unwind: {
		path: '$Nodes',
		preserveNullAndEmptyArrays: true // 空的数组也拆分
	}
},

{
	$lookup: { // 左连接
		from: 'P_DaaSProcessNodeConfig',
		let: {
			processId: '$id',
			fromNodeId: '$Nodes.targetId'
		},
		pipeline: [{
			$match: {
				$expr: {
					$and: [{
						$eq: ["$processId", "$$processId"]
					},
					{
						$eq: ["$pNodeId", "$$fromNodeId"]
					}]
				},
			},
		},
		{
			$project: {
				pNodeName: 1,
				pNodeType: 1,
				sqlCode: 1,
				dbConnId: 1,
				tableName: 1,
			},
		},
		],
		as: 'FromNode'
	}
},

{
	$unwind: {
		path: '$FromNode',
		preserveNullAndEmptyArrays: true // 空的数组也拆分
	}
},
{
	$lookup: { // 左连接
		from: 'P_DaaSProcessNodeConfig',
		// 关联到外联表
		let: {
			processId: '$id'
		},
		pipeline: [{
			$match: {
				pNodeType: 'rdbTableNode',
				$expr: {
					$eq: ["$processId", "$$processId"]
				},
			},
		},
		{
			$sort: {
				pNodeId: 1,
			} // 第一个写入节点是目标表
		},
		{
			$limit: 1,
		},
		{
			$project: {
				pNodeName: 1,
				pNodeType: 1,
				sqlCode: 1,
				dbConnId: 1,
				tableName: 1,
			},
		}],
		as: 'ToNode'
	},
},
{
	$unwind: {
		path: '$ToNode',
		preserveNullAndEmptyArrays: true // 空的数组也拆分
	}
},
{
	$lookup: { // 左连接
		from: 'P_AppMenuItemConfig',
		// 关联到外联表
		localField: 'categoryId',
		// 本表关联的字段
		foreignField: 'nodeId',
		// 外联表关联的字段
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
		from: 'P_AppMenuItemConfig',
		// 关联到外联表
		let: {
			parNodeId: '$Category2.parentNodeId',
			parCateId: '$Category2.categoryId'
		},
		pipeline: [{
			$match: {
				$expr: {
					$and: [{
						$eq: ["$nodeId", "$$parNodeId"]
					},
					{
						$eq: ["$categoryId", "$$parCateId"]
					}]
				},
			},
		},
		],
		as: 'Category1',
	},
},
{
	$unwind: { // 拆分子数组
		path: '$Category1',
		preserveNullAndEmptyArrays: true // 空的数组也拆分
	}
},
{
	$lookup: { // 左连接
		from: 'P_DaaSSchedulerRuleConfig',
		// 关联到P_DaaSSchedulerRuleConfig表
		localField: 'expression',
		// P_DaaSProcessModelConfig 表关联的字段
		foreignField: 'ruleId',
		// P_DaaSSchedulerRuleConfig 表关联的字段
		as: 'expressions'
	}
},
{
	$unwind: { // 拆分子数组
		path: '$expressions',
		preserveNullAndEmptyArrays: true // 空的数组也拆分
	}
},
{
	$lookup: { // 左连接
		from: 'P_DataSourceConfig',
		// 关联到P_DataSourceConfig表
		localField: 'FromNode.dbConnId',
		// P_DaaSProcessModelConfig 表关联的字段
		foreignField: 'configId',
		// P_DataSourceConfig 表关联的字段
		as: 'FromDataSource'
	}
},
{
	$unwind: { // 拆分子数组
		path: '$FromDataSource',
		preserveNullAndEmptyArrays: true // 空的数组也拆分
	}
},
{
	$lookup: { // 左连接
		from: 'P_DataSourceConfig',
		// 关联到P_DataSourceConfig表
		localField: 'ToNode.dbConnId',
		// P_DaaSProcessModelConfig 表关联的字段
		foreignField: 'configId',
		// P_DataSourceConfig 表关联的字段
		as: 'ToDataSource'
	}
},
{
	$unwind: { // 拆分子数组
		path: '$ToDataSource',
		preserveNullAndEmptyArrays: true // 空的数组也拆分
	}
},
{ // 映射，去掉这部分可以看到全部信息
	$project: {
		_id: 0,
		id: 1,
		'流程名称': '$configName',
		'流程类型': {
			$switch: {
				branches: [{
				case:
					{
						$eq:
						['$Category1.categoryId', 'daas.routerflow']
					},
					then: '数据流程'
				},
				{
				case:
					{
						$eq:
						['$Category1.categoryId', 'daas.datamodel']
					},
					then: '数据模型'
				},
				],
			default:
				'未知'
			},
		},
		'一级菜单': '$Category1.menuName',
		'二级菜单': '$Category2.menuName',
		'源头数据源名称': '$FromDataSource.configName',
		'源头数据源类型': '$FromDataSource.categoryId',
		'源头数据源链接': '$FromDataSource.jdbcUrl',
		'源头表SQL': '$FromNode.sqlCode',
		'源头表名': '$FromNode.tableName',
		'目标数据源名称': '$ToDataSource.configName',
		'目标数据源类型': '$ToDataSource.categoryId',
		'目标数据源链接': '$ToDataSource.jdbcUrl',
		'目标表名': '$ToNode.tableName',
		'创建流程时间': '$createTime',
		'最后修改时间': '$editTime',
		'调度机器': '$runServerId',
		'调度规则': "$expressions.expression",
		'调度名称': "$expressions.ruleName",
		'备注': '$remark',
		'负载均衡策略': '$LoadBalanceId',
		'运行状态': {
			$switch: {
				branches: [
					{ case: { $eq: ['$state', '0'] }, then: '手动' },
					{ case: { $eq: ['$state', '1'] }, then: '自动' },
					{ case: { $eq: ['$state', '2'] }, then: 'API' },
				],
			default:
				'未知'
			},
		}
	}
},
{
	$sort: {
		'流程创建时间': -1
	}
}])

// 如需推荐改名，则替换本行
processLists // 需替换行
```

```js
// 如需推荐改名，则将本行替换上脚本最后一行
var config = {
	connectorToken: '->',
	divideToken: ' @ ',
}
processLists.map(e => {
	var fromDataSourceInfos = e['源头数据源名称'] || ''
	var toDataSourceInfos = e['目标数据源名称'] || ''
	var f = JSON.parse(JSON.stringify(fromDataSourceInfos)).split('_')
	var t = JSON.parse(JSON.stringify(toDataSourceInfos)).split('_')
	if (f.length == 1 || t.length == 1) { // 如果是空数据源的就不对其进行推荐
		e['推荐流程名称'] = f.length == 1 ? '#无源头数据库源': '#无目标数据库源'
		return e
	}
	// 1.组建成：数据源->目标库 @
	ret = [f[1], f[0], config.connectorToken, t[1], t[0], config.divideToken]

	// 2.组建成：数据源->目标库 @ 流程名称 
	var pName = e['流程名称'].substr(e['流程名称'].indexOf(config.divideToken.trim()) + 1).trim() 
	ret.push(pName)

	// 3.组建成：数据源->目标库 @ 流程名称 目标表名
	if (e['流程名称'].indexOf(e['目标表名']) == -1) {
		ret.push(' ') 
		ret.push(e['目标表名'])
	}

	// 全英的情况认为是表名称
	// 4.组建成：数据源->目标库 @ 流程名称 目标表名(备注)
	if (e['备注'] && (e['备注'].match(/\s*\w+\s*/ && e['目标表名'] == null))) {
		ret = [...ret, '(', e['备注'].trim(), ')']
	}

	e['推荐流程名称'] = ret.join('')
	//	printjson(e)
	return e
})
```

