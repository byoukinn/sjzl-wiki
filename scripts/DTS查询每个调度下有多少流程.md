## 脚本介绍

该脚本通过查询DTS每个调度下有多少流程，来帮助判断哪个时间段流程过于集中，便于调整流程调度。

## 使用方法

连接至DTS数据库mongodb，要求用户空间为`Restcloud_DTS`，将以下内容全选复制直接执行即可观察结果。

## 查询每个调度下有几个流程的脚本内容
```js
//查询DTS每个调度下有多少流程
db.P_DaaSProcessModelConfig.aggregate([
	{	$match: { state:'1' }	},//条件判断：只筛选了调度自动状态的
    {
        $lookup: { // 左连接
          from: 'P_DaaSSchedulerRuleConfig', // 关联到P_DaaSSchedulerRuleConfig表
          localField: 'expression', // P_DaaSProcessModelConfig 表关联的字段
          foreignField: 'ruleId', // P_DaaSSchedulerRuleConfig 表关联的字段
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
        $group: {
            _id: '$expressions.ruleName',
            count:{$sum:1}
        }
     }
]);
```

## 查询每个调度下对应流程情况的脚本内容
```js
//详细数据
db.P_DaaSProcessModelConfig.aggregate([
    {
    	$match:{state:'1'}
    },
    {
        $lookup: { // 左连接
            from: 'P_DaaSSchedulerRuleConfig', // 关联到P_DaaSSchedulerRuleConfig表
            localField: 'expression', // P_DaaSProcessModelConfig 表关联的字段
            foreignField: 'ruleId', // P_DaaSSchedulerRuleConfig 表关联的字段
            as: 'expressions'}
        }, 
    {
        $unwind: { // 拆分子数组
            path: '$expressions',
            preserveNullAndEmptyArrays: true // 空的数组也拆分
        }
	}, 
    {
        $addFields: {  
            ruleId: "$expressions.ruleId",
            ruleName:"$expressions.ruleName",
            expression:"$expressions.expression"
        }//将左联表字段加到原表显示
    },
    {
        $project: {
            _id: 1,
            configId: 1,
            configName: 1,
            ruleId: 1,
            ruleName: 1,
            expression: 1
        }
	}
]);
```
