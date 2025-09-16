# Issue #56: 微信支付集成技术分析

## 概述

本文档分析Issue #56微信支付集成的技术实现方案，基于现有支付宝集成架构，设计完整的微信支付功能实现路径。

## 任务要求分析

根据 `.claude/epics/backend/56.md` 的要求：

### 验收标准
- [x] 微信支付创建 - POST /api/payments/wechat/create
- [x] 微信支付回调 - POST /api/payments/wechat/callback
- [x] 微信支付SDK集成 - 官方SDK集成
- [x] 微信支付签名验证 - 安全签名机制
- [x] 支付状态同步
- [x] 统一下单接口
- [x] 支付结果通知
- [x] 单元测试覆盖率>90%

### 技术要求
- 微信支付官方SDK
- JSAPI/APP/H5支付
- 签名算法实现
- 证书管理
- 异步通知处理

## 现有架构分析

### 支付宝实现架构回顾

基于对现有支付宝实现的分析（`/epic-backend/backend/app/`），现有架构包含：

1. **数据模型层** (`models/payment.py`)
   - `PaymentConfig`: 通用支付配置模型
   - `AlipayOrder`: 支付宝订单模型
   - `AlipayCallback`: 支付宝回调记录模型
   - `PaymentLog`: 支付操作日志模型

2. **Schema层** (`schemas/payment.py`)
   - 请求/响应模型定义
   - 数据验证和序列化
   - 异常类定义

3. **服务层** (`services/alipay_service.py`)
   - `AlipayService`: 核心业务逻辑
   - 支付创建、回调处理、状态查询
   - 签名验证和日志记录

4. **API层** (`api/v1/alipay.py`)
   - REST API端点定义
   - 请求处理和错误处理
   - 异步回调处理

## 微信支付技术特点分析

### 与支付宝的主要差异

| 特性 | 支付宝 | 微信支付 |
|------|--------|----------|
| 数据格式 | Form表单 | XML格式 |
| 签名算法 | RSA/RSA2 | MD5/HMAC-SHA256 |
| 金额单位 | 元 | 分 |
| 支付方式 | QUICK_WAP_PAY, FAST_INSTANT_TRADE_PAY | JSAPI, APP, MWEB, NATIVE |
| 回调响应 | "success"文本 | XML格式 |
| 用户标识 | 无特殊要求 | JSAPI需要openid |

### 技术挑战

1. **XML处理**: 需要XML解析和构建能力
2. **签名算法**: 实现HMAC-SHA256签名验证
3. **证书管理**: 商户证书和平台证书管理
4. **多支付方式**: 不同支付方式的参数差异
5. **openid获取**: JSAPI支付需要用户openid

## 架构设计

### 数据模型设计

#### 1. WechatPayOrder模型
```python
class WechatPayOrder(BaseModel):
    __tablename__ = "wechat_pay_orders"

    # 商户订单号（系统内部订单号）
    out_trade_no = Column(String(32), nullable=False, unique=True, comment="商户订单号")

    # 微信支付订单号
    transaction_id = Column(String(32), nullable=True, comment="微信支付订单号")

    # 商品描述
    body = Column(String(128), nullable=False, comment="商品描述")

    # 商品详情
    detail = Column(JSON, nullable=True, comment="商品详情")

    # 附加数据
    attach = Column(String(127), nullable=True, comment="附加数据")

    # 标价金额（分）
    total_fee = Column(Integer, nullable=False, comment="标价金额")

    # 货币类型
    fee_type = Column(String(16), nullable=False, default="CNY", comment="货币类型")

    # 交易类型
    trade_type = Column(String(16), nullable=False, comment="交易类型")  # JSAPI, NATIVE, APP, MWEB

    # 用户标识（JSAPI必须）
    openid = Column(String(128), nullable=True, comment="用户标识")

    # 商品标签
    goods_tag = Column(String(32), nullable=True, comment="商品标签")

    # 指定支付方式
    limit_pay = Column(String(32), nullable=True, comment="指定支付方式")

    # 用户ID
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False, comment="用户ID")

    # 支付状态
    status = Column(String(20), nullable=False, default="NOTPAY", comment="支付状态")

    # 支付时间
    time_end = Column(DateTime, nullable=True, comment="支付完成时间")

    # 预支付交易会话标识
    prepay_id = Column(String(64), nullable=True, comment="预支付ID")

    # 二维码链接
    code_url = Column(String(256), nullable=True, comment="二维码链接")

    # 支付跳转链接
    mweb_url = Column(String(256), nullable=True, comment="H5支付链接")
```

#### 2. WechatPayCallback模型
```python
class WechatPayCallback(BaseModel):
    __tablename__ = "wechat_pay_callbacks"

    # 关联的订单ID
    order_id = Column(Integer, ForeignKey("wechat_pay_orders.id"), nullable=False, comment="订单ID")

    # 返回状态码
    return_code = Column(String(16), nullable=False, comment="返回状态码")

    # 返回信息
    return_msg = Column(String(128), nullable=True, comment="返回信息")

    # 业务结果
    result_code = Column(String(16), nullable=False, comment="业务结果")

    # 错误代码
    err_code = Column(String(32), nullable=True, comment="错误代码")

    # 错误描述
    err_code_des = Column(String(128), nullable=True, comment="错误描述")

    # 微信支付订单号
    transaction_id = Column(String(32), nullable=True, comment="微信支付订单号")

    # 商户订单号
    out_trade_no = Column(String(32), nullable=False, comment="商户订单号")

    # 支付完成时间
    time_end = Column(String(14), nullable=True, comment="支付完成时间")

    # 原始XML数据
    raw_data = Column(Text, nullable=False, comment="原始XML数据")

    # 验签结果
    verify_result = Column(Boolean, nullable=False, default=False, comment="验签结果")

    # 处理状态
    process_status = Column(String(20), nullable=False, default="pending", comment="处理状态")

    # 处理时间
    processed_at = Column(DateTime, nullable=True, comment="处理时间")
```

#### 3. 复用PaymentConfig
微信支付配置复用现有`PaymentConfig`模型，`platform`字段设为"wechat"：

```python
config_data = {
    "appid": "微信支付应用ID",
    "mch_id": "商户号",
    "key": "商户密钥",
    "notify_url": "异步通知地址",
    "cert_path": "商户证书路径",
    "key_path": "商户私钥路径",
    "sandbox": "是否沙箱环境"
}
```

### API端点设计

#### 1. 创建支付订单
**POST /api/payments/wechat/create**

请求参数：
```python
class WechatPayCreateRequest(BaseModel):
    body: str = Field(..., max_length=128, description="商品描述")
    total_fee: int = Field(..., gt=0, description="支付金额（分）")
    trade_type: str = Field(..., description="交易类型 JSAPI|NATIVE|APP|MWEB")
    openid: Optional[str] = Field(None, description="用户openid（JSAPI必须）")
    detail: Optional[str] = Field(None, description="商品详情")
    attach: Optional[str] = Field(None, description="附加数据")
    goods_tag: Optional[str] = Field(None, description="商品标签")
    limit_pay: Optional[str] = Field(None, description="指定支付方式")
```

响应参数：
```python
class WechatPayCreateResponse(BaseModel):
    out_trade_no: str = Field(..., description="商户订单号")
    prepay_id: Optional[str] = Field(None, description="预支付ID")
    code_url: Optional[str] = Field(None, description="扫码支付链接")
    mweb_url: Optional[str] = Field(None, description="H5支付链接")
    status: str = Field(..., description="订单状态")
```

#### 2. 支付回调处理
**POST /api/payments/wechat/callback**

- 接收XML格式的回调数据
- 验证签名
- 更新订单状态
- 返回XML格式响应

#### 3. 查询支付状态
**GET /api/payments/wechat/status/{order_id}**

#### 4. 验证支付结果
**POST /api/payments/wechat/verify**

#### 5. 获取支付配置
**GET /api/payments/wechat/config**

### 签名验证机制

#### HMAC-SHA256签名实现
```python
def generate_wechat_signature(params: Dict[str, str], key: str) -> str:
    """生成微信支付签名"""
    # 1. 过滤空值和sign字段
    filtered_params = {k: str(v) for k, v in params.items()
                      if v and k != "sign"}

    # 2. 按key排序
    sorted_params = sorted(filtered_params.items())

    # 3. 拼接字符串
    sign_string = "&".join([f"{k}={v}" for k, v in sorted_params])
    sign_string += f"&key={key}"

    # 4. HMAC-SHA256签名
    signature = hmac.new(
        key.encode('utf-8'),
        sign_string.encode('utf-8'),
        hashlib.sha256
    ).hexdigest().upper()

    return signature

def verify_wechat_signature(params: Dict[str, str], key: str) -> bool:
    """验证微信支付签名"""
    sign = params.get("sign", "")
    if not sign:
        return False

    expected_sign = generate_wechat_signature(params, key)
    return sign == expected_sign
```

#### XML处理工具
```python
def xml_to_dict(xml_data: str) -> Dict[str, str]:
    """XML转字典"""
    import xml.etree.ElementTree as ET
    root = ET.fromstring(xml_data)
    return {child.tag: child.text for child in root}

def dict_to_xml(data: Dict[str, str]) -> str:
    """字典转XML"""
    xml_parts = ["<xml>"]
    for key, value in data.items():
        xml_parts.append(f"<{key}><![CDATA[{value}]]></{key}>")
    xml_parts.append("</xml>")
    return "".join(xml_parts)
```

### 错误处理策略

#### 异常类设计
```python
class WechatPayException(PaymentException):
    """微信支付异常"""
    pass

class WechatSignatureException(WechatPayException):
    """微信支付签名异常"""
    pass

class WechatCallbackException(WechatPayException):
    """微信支付回调异常"""
    pass
```

#### 错误码映射
```python
WECHAT_ERROR_CODES = {
    "NOAUTH": "商户无此接口权限",
    "NOTENOUGH": "余额不足",
    "ORDERPAID": "商户订单已支付",
    "ORDERCLOSED": "订单已关闭",
    "SYSTEMERROR": "系统错误",
    "APPID_NOT_EXIST": "APPID不存在",
    "MCHID_NOT_EXIST": "MCHID不存在",
    "SIGNERROR": "签名错误",
    "LACK_PARAMS": "缺少参数"
}
```

## 并行执行计划

### Stream划分策略

#### Stream 1: 基础架构 (无依赖)
- **任务**: 数据模型创建
  - WechatPayOrder模型定义
  - WechatPayCallback模型定义
  - 数据库迁移脚本
- **估时**: 1天
- **并行度**: 可与其他Stream并行

#### Stream 2: 核心服务 (依赖Stream 1)
- **任务**: WechatPayService实现
  - 统一下单逻辑
  - 签名验证工具
  - XML处理工具
  - 状态查询逻辑
- **估时**: 2天
- **并行度**: 内部可部分并行

#### Stream 3: API层 (依赖Stream 1, 2)
- **任务**: API端点实现
  - wechat.py路由文件
  - 请求/响应Schema
  - 回调处理端点
- **估时**: 1.5天
- **并行度**: 各端点可并行开发

#### Stream 4: 测试 (依赖Stream 1, 2, 3)
- **任务**: 测试覆盖
  - 单元测试（models, services）
  - API集成测试
  - 回调模拟测试
  - 签名验证测试
- **估时**: 1天
- **并行度**: 测试用例可并行编写

### 依赖关系图
```
Stream 1 (基础架构)
    ↓
Stream 2 (核心服务) → Stream 3 (API层)
    ↓                    ↓
    Stream 4 (测试) ←----
```

### 并行执行优化
1. **数据模型与Schema** - 可同时进行
2. **工具类开发** - 签名、XML工具可独立开发
3. **API端点** - 创建、查询、回调端点可并行
4. **测试用例** - 可与开发同步进行

## 技术实现路径

### Phase 1: 环境准备 (0.5天)
1. 创建微信支付配置
2. 安装依赖包（如需要）
3. 准备沙箱测试环境

### Phase 2: 基础架构 (1天)
1. 创建数据模型
2. 编写数据库迁移
3. 定义基础Schema

### Phase 3: 核心服务 (2天)
1. 实现WechatPayService
2. 开发签名验证工具
3. 实现XML处理工具
4. 统一下单接口

### Phase 4: API集成 (1.5天)
1. 创建API端点
2. 实现回调处理
3. 集成错误处理

### Phase 5: 测试验证 (1天)
1. 编写单元测试
2. 沙箱环境测试
3. 性能测试

**总估时**: 5工作日 (符合任务要求)

## 风险评估与缓解

### 主要风险
1. **微信API兼容性** - 版本变更风险
   - 缓解：使用稳定版本API，定期检查更新

2. **XML处理复杂性** - 解析错误风险
   - 缓解：充分测试，使用成熟XML库

3. **签名验证准确性** - 签名错误导致支付失败
   - 缓解：严格按照官方文档，多环境验证

4. **回调安全性** - 伪造回调风险
   - 缓解：严格签名验证，IP白名单

5. **证书管理** - 证书过期风险
   - 缓解：监控证书有效期，自动告警

### 测试策略
1. **单元测试**: 覆盖率>90%
2. **集成测试**: 沙箱环境全流程测试
3. **压力测试**: 模拟高并发回调
4. **安全测试**: 签名验证和防重放攻击

## 成功标准

### 功能完整性
- ✅ 支持JSAPI、APP、H5、Native四种支付方式
- ✅ 完整的回调处理和状态同步
- ✅ 签名验证和安全机制
- ✅ 错误处理和日志记录

### 质量标准
- ✅ 单元测试覆盖率>90%
- ✅ 代码符合项目规范
- ✅ 性能满足并发要求
- ✅ 安全通过渗透测试

### 维护性
- ✅ 代码结构清晰，易于扩展
- ✅ 完整的文档和注释
- ✅ 与现有架构一致
- ✅ 便于问题排查

## 结论

微信支付集成基于现有支付宝架构，通过合理的设计和并行开发策略，可以在5个工作日内完成高质量的实现。关键成功因素包括：

1. **架构复用**: 最大化复用现有支付架构
2. **并行开发**: 通过Stream划分实现并行开发
3. **质量保证**: 全面的测试覆盖和质量验证
4. **安全优先**: 重点关注签名验证和回调安全
5. **渐进实施**: 分阶段实施，降低风险

该方案为微信支付功能的成功实施提供了完整的技术路线图和实施指南。