# 测试流程的应用

## 需求评审
前提:提前阅读需求文档,记录疑惑点

目的:知道有什么功能,规则是什么,最终各部门理解一致.
## 计划编写
    测什么
    怎么测
    谁来测
    重点关注
        准入标准:研发提测标准,什么时候可以开始测试
            业务能跑通:P0
        准出标准:什么时候结束测试
            数据化:用例(100%) \缺陷(解决率:S0 100%,S1:100% S2\S3 :95%)

## 设计用例
先设计业务用例,后设计功能模块用例
## 用例执行
    按优先级(推荐)
        前提:写用例的时候标注清楚优先级并且明确优先级的定义
        P0:最高级别.
    按顺序执行
## 缺陷管理
    提交时间:用例执行失败的第一时间
    注意事项:
        唯一性
        可复现(明确复现步骤\问题发生时间\日志截图)
        注明版本号
## 测试报告