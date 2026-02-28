# 开放API接口调用指南

## 文档版本记录

| 版本 | 日期       | 修改说明 | 维护人员     |
| :--- | :--------- | :------- | :----------- |
| V1.0 | 2024-xx-xx | 初始版本 | 后端开发团队 |

------

## 1. 开放API接口调用

### 1.1 接口调用前提

在调用任何开放API接口之前，开发者必须完成以下准备工作：

#### 1.1.1 获取凭证信息

| 凭证类型    | 获取方式                 | 说明                         |
| :---------- | :----------------------- | :--------------------------- |
| **API-Key** | 联系近停视界工作人员申请 | 用于标识调用方身份的唯一密钥 |
| **RSA公钥** | 联系近停视界工作人员获取 | 用于加密API-Key的RSA公钥     |

> **重要提示**：以上凭证信息需通过官方渠道联系近停视界工作人员获取，请勿使用文档中的示例值进行生产环境调用。

#### 1.1.2 加密方法说明

本接口采用RSA非对称加密算法对API-Key进行加密，加密后的密文需放置在请求头中。

**加密要求：**

- **加密算法**：RSA
- **填充方式**：PKCS1Padding（RSA/ECB/PKCS1Padding）
- **字符编码**：UTF-8
- **输出格式**：Base64编码字符串

**加密流程：**

1. 获取API-Key明文（字符串形式）
2. 使用提供的RSA公钥对API-Key进行加密
3. 将加密结果进行Base64编码
4. 将最终密文放入请求头 `JTSJ-API-Key-Encrypted` 中

> **注意事项**：
>
> - 每次请求都需要重新生成加密密文
> - 加密结果会因实现方式略有差异，只要服务端能正确解密即可
> - 建议在正式调用前先进行加密验证

------

## 2. 儿童眼轴趋势预测接口

### 2.1 接口概述

根据儿童的基础信息（年龄、眼轴长度、角膜曲率等），获取眼轴发展趋势的预测数据。

#### 2.1.1 基本信息

| 项目             | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| **接口名称**     | 儿童眼轴趋势预测                                             |
| **请求方式**     | POST                                                         |
| **请求地址**     | `https://store.myopia88.com/dev-api/system/openapi/childArchives/eyeAxisPredict` |
| **Content-Type** | `application/json`                                           |

#### 2.1.2 接口特征

- **鉴权方式**：请求头携带RSA加密后的API-Key
- **数据格式**：请求/响应均为JSON格式
- **字符编码**：UTF-8

------

### 2.2 请求说明

#### 2.2.1 请求头（Headers）

| 参数名                 | 必填 | 类型   | 说明                     |
| :--------------------- | :--- | :----- | :----------------------- |
| JTSJ-API-Key-Encrypted | 是   | String | RSA加密后的API-Key密文   |
| Content-Type           | 是   | String | 固定值：application/json |

#### 2.2.2 请求体（Body）

| 字段名    | 类型   | 必填 | 说明                                  | 示例值 |
| :-------- | :----- | :--- | :------------------------------------ | :----- |
| age       | String | 是   | 儿童年龄（支持格式：X岁 或 X岁X个月） | 9岁    |
| eyeAxisOd | String | 是   | 右眼眼轴长度(mm)                      | 23.45  |
| eyeAxisOs | String | 是   | 左眼眼轴长度(mm)                      | 23.10  |
| odK1      | String | 是   | 右眼K1值（角膜曲率）                  | 43.25  |
| odK2      | String | 是   | 右眼K2值（角膜曲率）                  | 44.10  |
| osK1      | String | 是   | 左眼K1值（角膜曲率）                  | 43.50  |
| osK2      | String | 是   | 左眼K2值（角膜曲率）                  | 44.30  |

#### 2.2.3 请求示例

json

```
{
    "age": "15岁",
    "eyeAxisOd": "23.45",
    "eyeAxisOs": "23.10",
    "odK1": "43.25",
    "odK2": "44.10",
    "osK1": "43.50",
    "osK2": "44.30"
}
```



------

### 2.3 响应说明

#### 2.3.1 响应参数

| 字段名 | 类型    | 说明                |
| :----- | :------ | :------------------ |
| code   | Integer | 状态码（200：成功） |
| msg    | String  | 响应消息            |
| data   | Object  | 返回数据            |

#### 2.3.2 data数据说明

| 字段名             | 类型   | 说明                              |
| :----------------- | :----- | :-------------------------------- |
| sysAgeVos          | Array  | 年龄序列信息                      |
| ├─ ageMonth        | String | 年龄（X岁X个月格式）              |
| ├─ agePoint        | String | 年龄（小数格式）                  |
| ├─ alGrowthValue   | Double | 防控AL生理眼轴增长值              |
| └─ noAlGrowthValue | Double | 不防控AL生理眼轴增长值            |
| odAlmyoList        | Array  | 右眼 - 近视临界眼轴曲线数据       |
| odAlacceptList     | Array  | 右眼 - 防控优秀线数据             |
| odNoAlacceptList   | Array  | 右眼 - 不防控线（人群平均增速线） |
| odAlhealthList     | Array  | 右眼 - 健康远储眼轴曲线数据       |
| osAlmyoList        | Array  | 左眼 - 近视临界眼轴曲线数据       |
| osAlacceptList     | Array  | 左眼 - 防控优秀线数据             |
| osNoAlacceptList   | Array  | 左眼 - 不防控线（人群平均增速线） |
| osAlhealthList     | Array  | 左眼 - 健康远储眼轴曲线数据       |
| odMin              | Double | 右眼所有预测值的最小值            |
| odMax              | Double | 右眼所有预测值的最大值            |
| osMin              | Double | 左眼所有预测值的最小值            |
| osMax              | Double | 左眼所有预测值的最大值            |
| odCrossPointAge    | String | 右眼交叉点年龄                    |
| osCrossPointAge    | String | 左眼交叉点年龄                    |
| odNoAlacceptValue  | String | 右眼18岁时不防控的值              |
| osNoAlacceptValue  | String | 左眼18岁时不防控的值              |

#### 2.3.3 响应示例

json

```
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "sysAgeVos": [
            {
                "ageMonth": "17岁8个月",
                "agePoint": "17.67",
                "alGrowthValue": 0.008,
                "noAlGrowthValue": 0.02
            }
        ],
        "odAlmyoList": [23.98, 23.99, 24.0, 24.01, 24.03],
        "odAlacceptList": [23.45, 23.46, 23.47, 23.48, 23.49],
        "odNoAlacceptList": [23.45, 23.46, 23.47, 23.48, 23.49],
        "odAlhealthList": [23.77, 23.79, 23.8, 23.81, 23.83],
        "osAlmyoList": [23.89, 23.9, 23.91, 23.93, 23.94],
        "osAlacceptList": [23.1, 23.11, 23.12, 23.13, 23.14],
        "osNoAlacceptList": [23.1, 23.11, 23.12, 23.13, 23.14],
        "osAlhealthList": [23.69, 23.7, 23.71, 23.73, 23.74],
        "odMin": 23.45,
        "odMax": 24.03,
        "osMin": 23.1,
        "osMax": 23.94,
        "odCrossPointAge": null,
        "osCrossPointAge": null,
        "odNoAlacceptValue": "23.49",
        "osNoAlacceptValue": "23.14"
    }
}
```



------

### 2.4 调用示例（CURL）

bash

```
curl --location --request POST 'https://store.myopia88.com/dev-api/system/openapi/childArchives/eyeAxisPredict' \
--header 'JTSJ-API-Key-Encrypted: [此处填写RSA加密后的密文]' \
--header 'Content-Type: application/json' \
--data-raw '{
    "age": "5岁11个月",
    "eyeAxisOd": "23.15",
    "eyeAxisOs": "23.10",
    "odK1": "43.254",
    "odK2": "44.10",
    "osK1": "43.50",
    "osK2": "44.30"
}'
```



------

### 2.5 错误码说明

| 状态码 | 错误码 | 说明           | 处理建议                       |
| :----- | :----- | :------------- | :----------------------------- |
| 200    | -      | 成功           | 正常解析返回数据               |
| 400    | 10001  | 参数错误       | 检查请求参数格式和必填项       |
| 401    | 10002  | API-Key无效    | 检查加密是否正确或联系工作人员 |
| 401    | 10003  | API-Key已过期  | 联系工作人员重新获取           |
| 403    | 10004  | IP受限         | 确认IP是否在白名单中           |
| 429    | 10005  | 调用频率过高   | 降低调用频率                   |
| 500    | 20001  | 服务器内部错误 | 联系工作人员处理               |

------

## 3. 附录

### 3.1 注意事项

1. **凭证安全**：API-Key是重要凭证，请妥善保管，勿泄露
2. **加密时效**：建议每次请求重新生成加密密文
3. **年龄格式**：严格按照"X岁"或"X岁X个月"格式传入
4. **数据精度**：眼轴和K值建议保留2-3位小数
5. **调用频率**：请遵守接口调用频率限制

### 3.2 技术支持

如遇问题，请联系近停视界工作人员：

- 获取API-Key和RSA公钥
- 接口调用问题咨询
- 错误码排查协助
- 其他技术支持需求