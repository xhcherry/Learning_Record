# web抓包与测试报告

## 网络相关知识介绍

### 响应
    含义:服务器向客户端返回数据的过程
    组成
        响应行:协议/协议版本号 响应状态码 状态描述
            (面试题)响应状态码:
                2XX : 成功
                3XX : 重定向
                4XX : 客户端错误
                5XX : 服务器错误
        响应头: 服务器的属性信息
        响应体: 服务器返回的结果(图片\HTML\JSON\txt等等)
            JSON:{'name1':'value1'}