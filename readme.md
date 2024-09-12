
# 开发文档

## 目录

- [开发前必读](#开发前必读)
    - [简介](#简介)
    - [通用说明](#通用说明)
        - [签名规则](#签名规则)
    - [接口文档](#接口文档)
        - [通用错误接口](#通用错误接口)
        - [玩家登录](#玩家登录)
        - [转入转出](#转入转出)
        - [游戏列表](#游戏列表)
        - [进入游戏](#进入游戏)
        - [转账订单](#转账订单)
        - [游戏注单](#游戏注单)
        - [玩家信息](#玩家信息)
        - [转账订单确认](#转账订单确认)
        - [玩家登出](#玩家登出)
        - [注单详情页](#注单详情页)
        - [转出](#转出)
        - [转入](#转入)
- [游戏](#游戏)
  - [游戏介绍](#游戏介绍)
- [语言](#语言)
- [币种](#币种)


# 开发前必读
## 通用说明


### 签名规则
该方式用于直接采用MD5签名的方式

#### 签名算法

##### 第一步
设所有发送或者接收到的数据为集合M，将集合M内非空参数值的参数按照参数名ASCII码从小到大排序（字典序），使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串stringA。

特别注意以下重要规则：
- 参数名ASCII码从小到大排序（字典序）；
- 参数名区分大小写；
- sign参数不参与签名，平台主动通知传送的sign参数不参与签名
- 接口可能增加字段，验证签名时必须支持增加的扩展字段

##### 第二步
在stringA最后拼接上key得到stringSignTemp字符串，并对stringSignTemp进行MD5运算，再将得到的字符串所有字符转换为小写，得到sign值。

举例：
假设传送的参数如下：
```
agent_id : 2
account  : dg888888
nickname : harry
type     : 1
ip       : 127.0.0.1
timestamp: 1706941836000
```

对参数按照key=value的格式，并按照参数名ASCII字典序排序如下：
```go
stringA = "account=dg888888&agent_id=2&ip=127.0.0.1&nickname=harry&timestamp=1706941836000&type=1";
```

拼接API密钥：
```go
stringSignTemp = stringA + "&key=api_key" //注：api_key为后台设置的密钥key
```

假设密钥是: B9697969749DA2FF0B0057F363641EDA
```go
// GO 
api_key        := "B9697969749DA2FF0B0057F363641EDA"
stringA        := "account=dg888888&agent_id=2&ip=127.0.0.1&nickname=harry&timestamp=1706941836000&type=1";
stringSignTemp := stringA + "&key=" + api_key
// account=dg888888&agent_id=2&ip=127.0.0.1&nickname=harry&timestamp=1706941836000&type=1&key=B9697969749DA2FF0B0057F363641EDA

 // 计算 MD5 哈希值
hash := md5.New()
hash.Write([]byte(stringSignTemp))
hashInBytes := hash.Sum(nil) // 这将返回一个长度为 16 的字节数组

// 将字节数组转换为十六进制字符串
hashString := strings.ToUpper(hex.EncodeToString(hashInBytes))
// fmt.Println(hashString)
// hashString == 718566FD5CBA5FC27D2E645ACFE5B010
```

```java
// JAVA
String apiKey         = "B9697969749DA2FF0B0057F363641EDA";
String stringA        = "account=dg888888&agent_id=2&ip=127.0.0.1&nickname=harry&timestamp=1706941836000&type=1";
String stringSignTemp = stringA + "&key=" + apiKey;

try {
    // Create MD5 Hash
    MessageDigest md = MessageDigest.getInstance("MD5");
    md.update(stringSignTemp.getBytes(StandardCharsets.UTF_8));
    byte[] hashInBytes = md.digest();

    // Convert byte array into signum representation
    StringBuilder sb = new StringBuilder();
    for (byte b : hashInBytes) {
        sb.append(String.format("%02x", b));
    }
    String hashString = sb.toString().toUpperCase(); // 转换为大写形式

    // Output the hashString
    // System.out.println(hashString);  // 718566FD5CBA5FC27D2E645ACFE5B010
} catch (NoSuchAlgorithmException e) {
    e.printStackTrace();
}
```        
```php
<?php
$apiKey = "B9697969749DA2FF0B0057F363641EDA";
$stringA = "account=dg888888&agent_id=2&ip=127.0.0.1&nickname=harry&timestamp=1706941836000&type=1";
$stringSignTemp = $stringA . "&key=" . $apiKey;

// 计算 MD5 哈希值并转换为大写形式
$hashString = strtoupper(md5($stringSignTemp));

echo $hashString; // 输出: 718566FD5CBA5FC27D2E645ACFE5B010
?>
```

# 接口文档
## 通用错误接口
### 错误码与错误字符串
| 参数名       | 类型    | 描述    |
|--------------|---------|---------|
| error_code   | int     | 错误码  |
| error_msg    | string  | 错误信息 |



### 错误码实际内容
| Error Code | Error Message                | 错误码 | 错误信息                   |
|:----------:|------------------------------|:-----:|----------------------------|
| 0          | Success                      | 0    | 成功                       |
| 1          | Parameter Type Error         | 1    | 参数类型错误               |
| 2          | Parameter Value Error        | 2    | 参数值错误                 |
| 3          | Agent Not Found              | 3    | 代理不存在                 |
| 4          | Agent Disabled               | 4    | 代理已禁用                 |
| 5          | Signature Error              | 5    | 签名错误                   |
| 6          | Account Not Found            | 6    | 账号不存在                 |
| 7          | Game Not Enabled             | 7    | 游戏未启用                 |
| 8          | Token Not Found or Expired   | 8    | Token不存在或已过期        |
| 9          | Player Disabled              | 9    | 玩家已禁用                 |
| 10         | No Available Game Domain     | 10   | 无可用游戏域名             |
| 11         | Trans In Out Can't be Zero   | 11   | 转账金额不可以是0          |
| 12         | Timestamp is Expire          | 12   | 毫秒时间戳超时             |
| 13         | No Exist Order               | 13   | 订单不存在                |
| 14         | Insert Order is Exist        | 14   | 插入的订单已经存在           |
| 15         | Not Enough Money             | 15   | 金额不足                     |
| 99         | Internal Error               | 99   | 内部错误                   |



## 玩家登录
### 接口概述
- 功能: 玩家登录
- 请求方式: POST
- 请求地址: /api/player/login
### 请求参数
| 参数名    | 类型   | 是否必须 | 描述                                 | 示例值         |
|-----------|--------|:--------:|--------------------------------------|---------------|
| account   | string |    是    | 玩家账号,未注册会直接注册              | p47_tx1002212 |
| agent_id  | int    |    是    | 运营商ID                             | 1             |
| ip        | string |    否    | 玩家IP                               | 127.0.0.1     |
| nickname  | string |    否    | 玩家昵称                             | 恭喜发财007    |
| timestamp | int64  |    是    | 发送请求的毫秒时间戳                   | 1706941836000 |
| type      | int    |    是    | 玩家类型 1-正常                      | 1             |
| sign      | string |    是    | 签名，详见签名规则                   |               |
``` json
{
    "account"  : "p47_tx1002212",
    "agent_id" : 1,
    "ip"       : "127.0.0.1",
    "nickname" : "恭喜发财007",
    "timestamp": 1706941836000,
    "type"     : 1,
    "sign"     : ""
}
```



### 响应参数
| 参数名               | 类型      | 描述        |
|-------------------|---------|-----------|
| token             | string  | 玩家token   |

### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg" : "ok",
    "data"      : {
        "token": "ddec96d2165e4f3e8a642057db116983"
    }
}
```

#### 请求失败

```json
{
    "error_code": 1,
    "error_msg" : "参数类型错误, agent_id的类型是int不是string",
    "data"      : {}
}
```

## 转入转出
### 接口概述
- 功能: 转入转出
- 请求方式: POST
- 请求地址: /api/trans/inout
### 请求参数
| 参数名    | 类型   | 是否必须 | 描述                     | 示例值             |
|-----------|--------|:--------:|--------------------------|--------------------|
| account   | string |    是    | 玩家账号                 | p47heuf32rhwi      |
| agent_id  | int64  |    是    | 运营商ID                 | 1                  |
| amount    | string |    是    | 转入转出金额             | 127.22             |
| remark    | string |    否    | 备注                     | 备注               |
| out_order | string |    是    | 三方订单号               | O2024012268732     |
| timestamp | int64  |    是    | 发送请求的毫秒时间戳      | 1706941836000      |
| type      | int    |    是    | 类型(必填) 1-转出 2-转入  | 1                  |
| sign      | string |    是    | 签名，详见签名规则       |                    |


```json
{
    "account"  : "p47heuf32rhwi",
    "agent_id" : 1,
    "amount"   : "127.22",
    "remark"   : "备注",
    "out_order": "O2024012268732",
    "timestamp": 1706941836000,
    "type"     : 1,
    "sign"     : ""
}
```



### 响应参数
| 参数名           | 类型     | 描述                |
|------------------|----------|---------------------|
| agent_id         | int64    | 运营商ID             |
| order            | string   | 订单号               |
| out_order        | string   | 三方订单号            |
| create_time      | int64    | 创建时间             |
| account          | string   | 玩家账号             |
| type             | int      | 玩家类型 1-正常       |
| amount           | string   | 转账金额             |
| before_amount    | string   | 转账前金额            |
| after_amount     | string   | 转账后金额            |
| type             | int      | 类型 1-转出 2-转入   |
| currency_id      | int      | 币种                |
| remark           | string   | 备注                |



### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg": "ok",
    "data": {
        "agent_id"     : 1,
        "order"        : "17178332560293803578",
        "out_order"    : "O2024012268732",
        "create_time"  : 1717830218000,
        "account"      : "official_144335",
        "account_type" : 1,
        "amount"       : "100.00",
        "before_amount": "0.00",
        "after_amount" : "100.00",
        "type"         : 2,
        "currency_id"  : 1,
        "remark"       : "备注"
    }
}
```

#### 请求失败

```json
{
    "error_code": 1,
    "error_msg" : "参数类型错误",
    "data"      : {}
}
```

## 游戏列表
### 接口概述
- 功能: 游戏列表
- 请求方式: POST
- 请求地址: /api/game/list
### 请求参数
| 参数名      | 类型   | 是否必须 | 描述               | 示例值          |
|-------------|--------|:--------:|--------------------|-----------------|
| agent_id    | int64  |    是    | 运营商ID           | 1               |
| page        | int64  |    是    | 页数               | 1               |
| page_size   | int64  |    是    | 每页条数           | 10              |
| timestamp   | int64  |    是    | 发送请求的毫秒时间戳 | 170694


```json
{
    "agent_id" : 1,
    "page"     : 1,
    "page_size": 10,
    "timestamp": 1706941836000,
    "sign"     : ""
}
```

### 响应参数
| 参数名      | 类型   | 描述         |
|-------------|:------:|--------------|
| page        | int64  | 页数         |    
| page_size   | int64  | 每页条数     |
| total       | int64  | 总条数       |
| total_page  | int64  | 总页数       |
| list        | array  | 列表数组     |


#### 数组内容
| 参数名        | 类型   | 描述                                               |
|---------------|:------:|----------------------------------------------------|
| game_id       | string | 游戏ID                                             |
| game_name     | string | 游戏名称                                           |
| status        | int    | 状态 1-正常 2-禁用 3-维护 4-隐藏                  |
| tag           | int    | 游戏标签 0-全部 1-新品 2-推荐 3-经典 4-人气 5-活跃 6-最近 |
| category      | int    | 游戏分类 1-热门游戏 2-新品游戏                     |
| game_type     | int    | 游戏类型 1-电游类                                  |
| volatile      | int    | 游戏波动 1-低 2-中 3-高                            |
| theme         | int    | 游戏主题 1-神话 2-魔幻 3-现代 4-探索 5-自然 6-战争 7-故事  |



### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg" : "ok",
    "data"      : {
        "page"      : 1,
        "page_size" : 10,
        "total"     : 37,
        "total_page": 4,
        "list"      : [
          {
            "game_id"  : "2",
            "game_name": "麻将胡了",
            "status"   : 1,
            "tag"      : 2,
            "category" : 1,
            "game_type": 1,
            "volatile" : 3,
            "theme"    : 6
          }
        ]
    }
}
```

#### 请求失败

```json
{
    "error_code": 1,
    "error_msg" : "参数类型错误",
    "data"      : {}
}
```

## 进入游戏
### 接口概述
- 功能: 进入游戏
- 请求方式: POST
- 请求地址: /api/enter/game
### 请求参数
| 参数名       | 类型      | 是否必须 | 描述                | 示例值                            |
|-------------|-----------|:--------:|---------------------|----------------------------------|
| agent_id    | int64     |    是    | 运营商ID             | 1                                |
| game_id     | int64     |    是    | 游戏ID               | 1                                |
| lang        | string    |    是    | 语系                 | zh-hans                          |
| timestamp   | int64     |    是    | 发送请求的毫秒时间戳 | 1706941836000                    |
| token       | string    |    是    | 玩家token (玩家注入接口获取) | ddec96d2165e4f3e8a642057db116983 |
| sign        | string    |    是    | 签名，详见签名规则   |                                  |


### 响应参数
| 参数名      | 类型     | 描述    |
|------------|--------|-------|
| play_url   | string | 游戏链接 |


### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg" : "ok",
    "data"      : {
        "play_url": "www.game.com/game.html"
    }
}
```

#### 请求失败

```json
{
    "error_code": 1,
    "error_msg" : "参数类型错误",
    "data"      : {}
}
```

## 转账订单
### 接口概述
- 功能: 转账订单
- 请求方式: POST
- 请求地址: /api/transfer/order
### 请求参数
| 参数名       |  类型  | 是否必须 | 描述                     | 示例值            |
|--------------|:------:|:--------:|--------------------------|-------------------|
| account      | string |    是    | 玩家账号                 | 1                 |
| agent_id     | int64  |    是    | 运营商ID                 | 1                 |
| end_time     | int64  |    是    | 结束毫秒时间戳           | 1                 |
| page         | int64  |    是    | 页数                     | 1                 |
| page_size    | int64  |    是    | 每页条数                 | 10                |
| start_time   | int64  |    是    | 开始毫秒时间戳           | 1                 |
| timestamp    | int64  |    是    | 发送请求的毫秒时间戳     | 1706941836000     |
| type         | int64  |    是    | 玩家类型 0-正常玩家 1-试玩玩家 | 1                 |
| sign         | string |    是    | 签名，详见签名规则       |                   |


### 响应参数
| 参数名            | 类型     | 描述             |
|----------------|:------:|----------------|
| page           | int64  | 页数             |    
| page_size      | int64  | 每页条数           |
| total          | int64  | 总条数            |
| total_page     | int64  | 总页数            |
| list           | array  | 列表数组           |

#### 数据内容
| 参数名         | 类型 | 描述 | 
| -------------| :--------:| -------------- |
| order         | string | 订单号            |
| out_order      | string | 三方订单号          |
| create_time    | int    | 创建时间           |
| account        | string | 玩家账号           |
| account_type   | string | 账号类型0-正常 1-试玩  |
| amount         | string | 转账金额           |
| before_amount  | string | 转账前金额          |
| after_amount   | string | 转账后金额          |
| type           | int    | 账变类型 1-转出 2-转入 |
| currency_id    | int    | 币种ID           |
| remark         | string | 备注             |


### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg" : "ok",
    "data"      : {
        "page"      : 1,
        "page_size" : 10,
        "total"     : 37,
        "total_page": 4,
        "list"      : [
          {
            "order"        : "20240102184048",
            "out_order"    : "O37432423",
            "create_time"  : 170590352900,
            "account"      : "P3sda3qs",
            "account_type" : 1,
            "amount"       : "1.00",
            "before_amount": "0.00",
            "after_amount" : "1.00",
            "type"         : 1,
            "currency_id"  : 1,
            "remark"       : "备注"
          }
        ]
    }
}
```


## 游戏注单
### 接口概述
- 功能: 游戏注单
- 请求方式: POST
- 请求地址: /api/game/orders
- 备注：单次最少获取1分钟区间内所有订单数据，最大不能超过60分钟，我方每分钟更新注单数据，请勿多次重复拉取
### 请求参数
| 参数名       | 类型     | 是否必须 | 描述               | 示例值           |
|--------------|----------|:--------:|--------------------|------------------|
| agent_id     | int64    |    是    | 运营商ID           | 1                |
| end_time     | int64    |    是    | 结束毫秒时间戳      | 1706941866000    |
| start_time   | int64    |    是    | 开始毫秒时间戳      | 1706941836000    |
| timestamp    | int64    |    是    | 发送请求的毫秒时间戳 | 1706941836000    |
| sign         | string   |    是    | 签名，详见签名规则  |                  |


### 响应参数
| 参数名           | 类型    | 描述                             |
|------------------|---------|----------------------------------|
| list             | array   | 列表数组                         |

#### 数据内容
| 参数名           | 类型   | 描述                             |
|------------------|:------:|----------------------------------|
| parent_bet_id    | string | 父单ID                           |
| ref_id           | string | 订单ID                           |
| round_id         | string | 回合ID                           |
| game_id          | string | 游戏ID                           |
| game_name        | string | 游戏名称                         |
| account          | string | 玩家账号                          |
| bet_time         | int64  | 下注时间毫秒时间戳               |
| start_time       | int64  | 开始时间毫秒时间戳, 定时类游戏使用 |
| end_time         | int64  | 结束时间毫秒时间戳, 定时类游戏使用 |
| bet_amount       | string | 下注金额,最多保留2位小数          |
| payout_amount    | string | 派彩金额,最多保留2位小数          |
| overage          | string | 输赢金额,最多保留2位小数          |
| status           | int    | 交易状态 1-未完成 2-已完成        |
| currency_id      | int    | 币种ID 具体可以参看币种表         |

假设玩家下注1.00元, 未中奖
```json
{
    "bet_amount"   : "1.00",
    "payout_amount": "0.00",
    "overage"      : "-1.00"
}
```

假设玩家下注1.00元, 中奖1元, 等于保本
```json
{
    "bet_amount"   : "1.00",
    "payout_amount": "1.00",
    "overage"      : "0.00"
}
```

假设玩家下注1.00元, 中奖4元, 等于赢了3元
```json
{
    "bet_amount"   : "1.00",
    "payout_amount": "4.00",
    "overage"      : "3.00"
}
```

### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg": "ok",
    "data": {
        "list": [
            {                
                "parent_bet_id": "17176457141975900008",
                "ref_id"       : "17176457141975900008",
                "round_id"     : "O374332423",
                "game_id"      : "2",
                "game_name"    : "麻将胡了",
                "account"      : "awt176263",
                "bet_time"     : 1706941836000,
                "start_time"   : 1706941836000,
                "end_time"     : 1706941836000,
                "bet_amount"   : "1.00",
                "payout_amount": "0.00",
                "overage"      : "-1.00",
                "status"       : 1,
                "currency_id"  : 1,
            },
            {                
                "parent_bet_id": "17176457141975900008",
                "ref_id"       : "17176457141976000009",
                "round_id"     : "O374332424",
                "game_id"      : "2",
                "game_name"    : "麻将胡了",
                "account"      : "awt176263",
                "bet_time"     : 1706941838000,
                "start_time"   : 1706941838000,
                "end_time"     : 1706941838000,
                "bet_amount"   : "1.00",
                "payout_amount": "4.00",
                "overage"      : "3.00",
                "status"       : 1,
                "currency_id"  : 1,
            }
        ]
    }
}

```
#### 请求失败

```json
{
    "error_code": 1,
    "error_msg" : "参数类型错误",
    "data"      : {}
}
```


## 玩家信息
### 接口概述
- 功能: 玩家信息
- 请求方式: POST
- 请求地址: /api/player/info
### 请求参数
| 参数名       | 类型   | 是否必须 | 描述               | 示例值           |
|--------------|--------|:--------:|--------------------|------------------|
| account      | string |    是    | 玩家账号           | p47heuf32rhwi    |
| agent_id     | int64  |    是    | 运营商ID           | 1                |
| timestamp    | int64  |    是    | 发送请求的毫秒时间戳 | 1706941836000    |
| sign         | string |    是    | 签名，详见签名规则  |                  |


### 响应参数
| 参数名           | 类型   | 描述                       |
|------------------|:------:|----------------------------|
| account          | string | 玩家账号                   |
| nickname         | string | 玩家昵称                   |
| balance          | string | 玩家余额                   |
| frozen_amount    | string | 玩家冻结金额               |
| is_online        | int    | 玩家是否在线               |
| account_type     | int    | 玩家类型 0-正常 1-试玩      |
| status           | int    | 状态 1-正常 2-冻结 3-封号  |


### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg":"ok",
    "data":{
        "account"      : "aa3242",
        "nickname"     : "dg",
        "balance"      : "100.22",
        "frozen_amount": "16.83",
        "is_online"    : 1,
        "account_type" : 1,
        "status"       : 1
    }
}
```

#### 请求失败

```json
{
    "error_code": 1,
    "error_msg" : "参数类型错误",
    "data"      : {}
}
```


## 玩家余额
### 接口概括
- 功能: 只获取玩家余额, 通常也可以使用上面的【玩家信息】接口代替
- 请求方式: POST
- 请求地址: /api/player/balance
### 请求参数
| 参数名       | 类型   | 是否必须 | 描述               | 示例值           |
|--------------|--------|:--------:|--------------------|------------------|
| account      | string |    是    | 玩家账号           | p47heuf32rhwi    |
| agent_id     | int64  |    是    | 运营商ID           | 1                |
| timestamp    | int64  |    是    | 发送请求的毫秒时间戳 | 1706941836000    |
| sign         | string |    是    | 签名，详见签名规则  |                  |




### 响应参数
| 参数名           | 类型   | 描述                       |
|------------------|:------:|----------------------------|
| balance          | string | 玩家余额                   |



### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg":"ok",
    "data":{
        "account"      : "aa3242",        
        "balance"      : "100.22",        
    }
}
```

#### 请求失败

```json
{
    "error_code": 1,
    "error_msg" : "参数类型错误",
    "data"      : {}
}
```





## 转账订单确认
### 接口概述
- 功能: 转账订单确认
- 请求方式: POST
- 请求地址: /api/transfer/check
### 请求参数
| 参数名     | 类型   | 是否必须 | 描述               | 示例值        |
|------------|--------|:--------:|--------------------|---------------|
| agent_id   | int64  |    是    | 运营商ID           | 1             |
| out_order  | string |    是    | 第三方订单号       | O38674837624  |
| timestamp  | int64  |    是    | 发送请求的时间戳   | 1626863144    |
| sign       | string |    是    | 签名，详见签名规则 |               |



### 响应参数
| 参数名           | 类型    | 描述                    |
|------------------|---------|-----------------------|
| error_code       | int     | 错误码                  |
| error_msg        | string  | 错误信息                |
| out_order        | string  | 第三方订单号             |
| status           | int     | 订单状态 1：完成 2：未完成 |
| amount           | string  | 转账金额                   |
| before_amount    | string  | 转账前金额                 |
| after_amount     | string  | 转账后金额                 |


### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg" : "ok",
    "data"      : {
        "out_order"    : "O348274783264",
        "status"       : 1,
        "amount"       : "16.83",
        "before_amount": "100.22",
        "after_amount" : "100.22"
    }
}
```

#### 请求失败

```json
{
    "error_code": 13,
    "error_msg" : "订单不存在",
    "data"      : {}
}
```


## 玩家登出
### 接口概述
- 功能: 玩家登出或者踢号
- 请求方式: POST
- 请求地址: /api/player/logout
### 请求参数
| 参数名    | 类型   | 是否必须 | 描述                                 | 示例值         |
|-----------|--------|:--------:|--------------------------------------|---------------|
| account   | string |    是    | 玩家账号                            | p47_tx1002212 |
| agent_id  | int    |    是    | 运营商ID                             | 1             |
| timestamp | int64  |    是    | 发送请求的毫秒时间戳                   | 1706941836000 |
| sign      | string |    是    | 签名，详见签名规则                   |               |
``` json
{
    "account"  : "HQ_28_hello8",
    "agent_id" : 1,    
    "timestamp": 1706941836000,
    "type"     : 1,
    "sign"     : ""
}
```



### 响应参数
| 参数名               | 类型      | 描述        |
|-------------------|---------|-----------|


### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg" : "ok",
    "data"      : { 
    }
}
```

#### 请求失败

```json
{
    "error_code": 1,
    "error_msg" : "参数类型错误, agent_id的类型是int不是string",
    "data"      : {}
}
```




## 注单详情页
### 接口概述
- 功能: 注单详情页的生产
- 请求方式: POST
- 请求地址: /api/order/detail
### 请求参数
| 参数名    | 类型   | 是否必须 | 描述                                 | 示例值         |
|-----------|--------|:--------:|--------------------------------------|---------------|
| parent_bet_id   | string |    是    | 注单的父单ID              | 17176457141975900008 |
| agent_id  | int    |    是    | 运营商ID                             | 1             |
| timestamp | int64  |    是    | 发送请求的毫秒时间戳                   | 1706941836000 |
| sign      | string |    是    | 签名，详见签名规则                   |               |
``` json
{
    "parent_bet_id": "17176457141975900008",
    "agent_id"     : 1,
    "timestamp"    : 1706941836000,
    "sign"         : ""
}
```



### 响应参数
| 参数名               | 类型      | 描述        |
|-------------------|---------|-----------|
| url               | string  | 详情的URL地址  |
| game_id           | int     | 游戏编号   |

### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg" : "ok",
    "data"      : {
        "url"    : "https://info.lgsofts.com/#/?gameid=5&order=005DBFB5AF6AC214AFE9",
        "game_id": 5
    }
}
```

#### 请求失败

```json
{
    "error_code": 1,
    "error_msg" : "参数类型错误, agent_id的类型是int不是string",
    "data"      : {}
}
```

 
 
## 转出
### 接口概述
- 功能: 转出功能(下分)
- 请求方式: POST
- 请求地址: /api/trans/out

```
/api/trans/out 和/api/trans/inout的区别在与:
如果账户有99.5, 请求转出100, 
/api/trans/inout接口会转出99.5，余额变成0
/api/trans/out接口会直接报错中止
```

### 请求参数
| 参数名    | 类型   | 是否必须 | 描述                     | 示例值             |
|-----------|--------|:--------:|--------------------------|--------------------|
| account   | string |    是    | 玩家账号                 | p47heuf32rhwi      |
| agent_id  | int64  |    是    | 运营商ID                 | 1                  |
| amount    | string |    是    | 转入转出金额             | 127.22             |
| remark    | string |    否    | 备注                     | 备注               |
| out_order | string |    是    | 三方订单号               | O2024012268732     |
| timestamp | int64  |    是    | 发送请求的毫秒时间戳      | 1706941836000      |
| sign      | string |    是    | 签名，详见签名规则       |                    |


```json
{
    "account"  : "p47heuf32rhwi",
    "agent_id" : 1,
    "amount"   : "100.00",
    "remark"   : "备注",
    "out_order": "LuckyGamingb51a7f36db414ace",
    "timestamp": 1706941836000,
    "sign"     : ""
}
```



### 响应参数
| 参数名           | 类型     | 描述                |
|------------------|----------|---------------------|
| agent_id         | int64    | 运营商ID             |
| order            | string   | 订单号               |
| out_order        | string   | 三方订单号            |
| create_time      | int64    | 创建时间             |
| account          | string   | 玩家账号             |
| type             | int      | 玩家类型 1-正常       |
| amount           | string   | 转账金额             |
| before_amount    | string   | 转账前金额            |
| after_amount     | string   | 转账后金额            |
| type             | int      | 类型 1-转出 2-转入   |
| currency_id      | int      | 币种                |
| remark           | string   | 备注                |



### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg": "ok",
    "data": {
        "agent_id"     : 1,
        "order"        : "17178332560293803578",
        "out_order"    : "O2024012268732",
        "create_time"  : 1717830218000,
        "account"      : "official_144335",
        "account_type" : 1,
        "amount"       : "100.00",
        "before_amount": "100.00",
        "after_amount" : "0.00",
        "type"         : 1,
        "currency_id"  : 1,
        "remark"       : "备注"
    }
}
```

#### 请求失败

```json
{
    "error_code": 15,
    "error_msg" : "Not Enough Money",
    "data"      : {}
}
```


 
## 转入
### 接口概述
- 功能: 转入功能(上分)
- 请求方式: POST
- 请求地址: /api/trans/in

```
/api/trans/in 和/api/trans/inout的区别在与:
少了一个type参数
```

### 请求参数
| 参数名    | 类型   | 是否必须 | 描述                     | 示例值             |
|-----------|--------|:--------:|--------------------------|--------------------|
| account   | string |    是    | 玩家账号                 | p47heuf32rhwi      |
| agent_id  | int64  |    是    | 运营商ID                 | 1                  |
| amount    | string |    是    | 转入转出金额             | 127.22             |
| remark    | string |    否    | 备注                     | 备注               |
| out_order | string |    是    | 三方订单号               | O2024012268732     |
| timestamp | int64  |    是    | 发送请求的毫秒时间戳      | 1706941836000      |
| sign      | string |    是    | 签名，详见签名规则       |                    |


```json
{
    "account"  : "p47heuf32rhwi",
    "agent_id" : 1,
    "amount"   : "127.22",
    "remark"   : "备注",
    "out_order": "LuckyGamingb51a7f36db414ace",
    "timestamp": 1706941836000,
    "sign"     : ""
}
```



### 响应参数
| 参数名           | 类型     | 描述                |
|------------------|----------|---------------------|
| agent_id         | int64    | 运营商ID             |
| order            | string   | 订单号               |
| out_order        | string   | 三方订单号            |
| create_time      | int64    | 创建时间             |
| account          | string   | 玩家账号             |
| type             | int      | 玩家类型 1-正常       |
| amount           | string   | 转账金额             |
| before_amount    | string   | 转账前金额            |
| after_amount     | string   | 转账后金额            |
| type             | int      | 类型 1-转出 2-转入   |
| currency_id      | int      | 币种                |
| remark           | string   | 备注                |



### 响应实例
#### 请求成功
```json
{
    "error_code": 0,
    "error_msg": "ok",
    "data": {
        "agent_id"     : 1,
        "order"        : "17178332560293803578",
        "out_order"    : "O2024012268732",
        "create_time"  : 1717830218000,
        "account"      : "official_144335",
        "account_type" : 1,
        "amount"       : "100.00",
        "before_amount": "0.00",
        "after_amount" : "100.00",
        "type"         : 2,
        "currency_id"  : 1,
        "remark"       : "备注"
    }
}
```

#### 请求失败

```json
{
    "error_code": 1,
    "error_msg" : "参数类型错误",
    "data"      : {}
}
```

# 其他参数
## 游戏
### 游戏介绍
| 游戏ID | 游戏名称          | 英文名称                  | 题材     | 简介                          |
|--------|-----------------|-------------------------|----------|-----------------------------|
| 2      | 麻将胡了          | Mahjong Ways            | 麻将     | 24小时麻将馆/10万倍             |
| 4      | 财神来了          | Cai Shen Ways           | 财神     | 中路连消百搭有机会赢更多         |
| 5      | 赏金猎人          | Bounty Hunter           | 西部     | 高达X40赢奖倍数                |
| 6      | 招财喵            | Lucky Neko              | 奇幻     | 赢特色招财猫符号，送递增奖金倍数  |
| 7      | 寻宝黄金城        | Treasures of Aztec      | 探险     | 多路连消百搭                    |
| 8      | 神鸟报恩          | Divine Bird Returns     | 越南神话 | 杨桃变金币，高达X16200          |
| 9      | 虎虎生财          | Fortune Tiger           | 奇幻     | 重新旋转和X10奖金倍数！         |
| 10     | 热带雨林          | Rain Forest             | 动物     | 无线卷轴递增奖金倍数             |
| 11     | 亡灵大盗          | Wild Bandito            | 奇幻     | 递增式奖金倍数和中轴的金框符号    |
| 12     | 山精水精          | Mountain Lord VS Sea Lord | 越南神话 | 多种玩法超高倍数，战斗奖金高高高！ |
| 13     | 淘金者            | Gold Rush               | 探险     | 收集符号触发多种模式，巨龙获得高额奖励！ |
| 14     | 冰雪大冲关        | The Great Icescape      | 动物     | 破冰行动赢取X50000倍投注奖励     |
| 15     | 金球射手          | Ultimate Striker        | 足球     | 编入多路连消百搭，增加奖金倍数最高X8000倍 |
| 16     | 众神宙斯          | Zeus Power Link         | 北欧神话 | Feature Multiplier symbols and Free Spins feature |
| 17     | 热血欧洲杯        | Passionate European Cup | 足球     | Feature a multipying Wild symbol and Free Spin collection prize pool mode |
| 18     | 狂欢音乐节        | Music Festival          | 音乐     | Feature increasing Multipliers and Bonus Game |
| 19     | 财神赐福        | CaiShen Fortune         | 财神     |  |

## 语言
### 语言与描述
| 语言名称 | 描述 | 
|--------------|---------------------|
|zh-hans  | 简体中文|
|vi       | 越南文|
|en       | 英语 |



## 币种
### 币种和币种ID描述
| 币种ID | 币种名称 | 币种缩写 | 
|--------------|-----------|----------|
|-1|    试玩币     | TC |
|1 |   人民币     | CNY |
|2 |   美元     | USD |
|3 |   欧元     | EUR |
|4 |   日元     | JPY |
|5 |   英镑     | GBP |
|6 |   加拿大元     | CAD |
|7 |   澳大利亚元     | AUD |
|8 |   瑞士法郎     | CHF |
|9 |   港币     | HKD |
|10|   迪拉姆     | AED |
|11|   菲律宾比索     | PHP |
|12|   越南盾     | VND |
|13|   瑞典克朗     | SEK |
|14|   挪威克朗     | NOK |
|15|   丹麦克朗     | DKK |
|16|   新加坡元     | SGD |
|17|   新西兰元     | NZD |
|18|   南非兰特     | ZAR |
|19|   巴西雷亚尔     | BRL |
|20|   俄罗斯卢布     | RUB |
|21|   印度卢比     | INR |
|22|   韩国元     | KRW |
|23|   土耳其里拉     | TRY |
|24|   波兰兹罗提     | PLN |
|25|   泰铢     | THB |
|26|   马来西亚林吉特     | MYR |
|27|   印尼盾     | IDR |
|28|   墨西哥比索     | MXN |
|29|   阿根廷比索     | ARS |
|30|   埃及镑     | EGP |
|31|   以色列新谢克尔     | ILS |
|32|   沙特里亚尔     | SAR |
|33|   卡塔尔里亚尔     | QAR |
|34|   约旦第纳尔     | JOD |
|35|   巴林第纳尔     | BHD |
|36|   阿曼里亚尔     | OMR |
|37|   科威特第纳尔     | KWD |
|38|   新台币     | TWD |
|39|   匈牙利福林     | HUF |
|40|   捷克克朗     | CZK |
|41|   罗马尼亚列伊     | RON |
|42|   哥伦比亚比索     | COP |
|43|   最大值限定     | MAX |