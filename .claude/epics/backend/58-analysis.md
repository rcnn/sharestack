# Issue #58: 支付状态管理技术实现方案

## 文档信息
- **创建时间**: 2025-09-16
- **任务编号**: Issue #58 / Task 037
- **分析版本**: v1.0
- **依赖任务**: Task 001 (基础架构), Task 055 (支付宝), Task 056 (微信), Task 057 (银联)

## 1. 技术分析

### 1.1 任务概述
实现统一的支付状态管理系统，整合支付宝、微信支付、银联支付三个平台，提供统一的状态查询、验证、退款和异常处理功能。

### 1.2 现有支付系统分析

#### 支付系统现状
根据对Task 055-057的分析，当前支付系统包含：

| 支付平台 | 数据模型 | 特点 | 状态管理 |
|----------|----------|------|----------|
| 支付宝 | AlipayOrder, AlipayCallback | Form表单、RSA签名、异步回调 | 独立状态管理 |
| 微信支付 | WechatPayOrder, WechatPayCallback | XML格式、HMAC-SHA256、多支付方式 | 独立状态管理 |
| 银联支付 | UnionPayOrder, UnionPayCallback | 网关+快捷、RSA签名、证书管理 | 独立状态管理 |

#### 架构挑战
1. **状态不统一**: 各平台状态定义和转换规则不同
2. **接口分散**: 缺乏统一的查询和管理接口
3. **监控困难**: 无法进行跨平台的支付监控和审计
4. **维护复杂**: 三套独立的状态管理逻辑

### 1.3 设计目标
- **统一抽象**: 提供统一的支付状态模型和接口
- **向后兼容**: 不破坏现有支付平台的独立功能
- **高可用性**: 支持状态同步、异常恢复和监控告警
- **可扩展性**: 便于接入新的支付平台

## 2. 架构设计

### 2.1 统一支付状态模型

#### 2.1.1 统一状态枚举
```python
from enum import Enum

class UnifiedPaymentStatus(Enum):
    """统一支付状态枚举"""
    PENDING = "pending"           # 待支付
    PROCESSING = "processing"     # 处理中
    SUCCESS = "success"           # 支付成功
    FAILED = "failed"             # 支付失败
    CANCELLED = "cancelled"       # 已取消
    REFUNDED = "refunded"         # 已退款
    PARTIAL_REFUNDED = "partial_refunded"  # 部分退款
    TIMEOUT = "timeout"           # 超时
    UNKNOWN = "unknown"           # 未知状态

class PaymentPlatform(Enum):
    """支付平台枚举"""
    ALIPAY = "alipay"
    WECHAT = "wechat"
    UNIONPAY = "unionpay"
```

#### 2.1.2 状态映射机制
```python
# 各平台状态到统一状态的映射
PLATFORM_STATUS_MAPPING = {
    PaymentPlatform.ALIPAY: {
        "WAIT_BUYER_PAY": UnifiedPaymentStatus.PENDING,
        "TRADE_SUCCESS": UnifiedPaymentStatus.SUCCESS,
        "TRADE_FINISHED": UnifiedPaymentStatus.SUCCESS,
        "TRADE_CLOSED": UnifiedPaymentStatus.CANCELLED,
        "TRADE_FAILED": UnifiedPaymentStatus.FAILED,
    },
    PaymentPlatform.WECHAT: {
        "NOTPAY": UnifiedPaymentStatus.PENDING,
        "USERPAYING": UnifiedPaymentStatus.PROCESSING,
        "SUCCESS": UnifiedPaymentStatus.SUCCESS,
        "PAYERROR": UnifiedPaymentStatus.FAILED,
        "CLOSED": UnifiedPaymentStatus.CANCELLED,
        "REFUND": UnifiedPaymentStatus.REFUNDED,
    },
    PaymentPlatform.UNIONPAY: {
        "00": UnifiedPaymentStatus.SUCCESS,     # 成功
        "01": UnifiedPaymentStatus.FAILED,      # 失败
        "02": UnifiedPaymentStatus.PROCESSING,  # 处理中
        "03": UnifiedPaymentStatus.CANCELLED,   # 已取消
    }
}
```

#### 2.1.3 状态机设计
```python
# 合法的状态转换规则
VALID_STATUS_TRANSITIONS = {
    UnifiedPaymentStatus.PENDING: [
        UnifiedPaymentStatus.PROCESSING,
        UnifiedPaymentStatus.SUCCESS,
        UnifiedPaymentStatus.FAILED,
        UnifiedPaymentStatus.CANCELLED,
        UnifiedPaymentStatus.TIMEOUT
    ],
    UnifiedPaymentStatus.PROCESSING: [
        UnifiedPaymentStatus.SUCCESS,
        UnifiedPaymentStatus.FAILED,
        UnifiedPaymentStatus.TIMEOUT
    ],
    UnifiedPaymentStatus.SUCCESS: [
        UnifiedPaymentStatus.REFUNDED,
        UnifiedPaymentStatus.PARTIAL_REFUNDED
    ],
    UnifiedPaymentStatus.FAILED: [],  # 终态
    UnifiedPaymentStatus.CANCELLED: [],  # 终态
    UnifiedPaymentStatus.REFUNDED: [],  # 终态
    UnifiedPaymentStatus.PARTIAL_REFUNDED: [UnifiedPaymentStatus.REFUNDED],
    UnifiedPaymentStatus.TIMEOUT: [],  # 终态
    UnifiedPaymentStatus.UNKNOWN: []  # 需要人工处理
}
```

### 2.2 统一支付模型

#### 2.2.1 UnifiedPayment 模型
```python
class UnifiedPayment(BaseModel):
    """统一支付记录模型"""
    __tablename__ = "unified_payments"

    # 主键和唯一标识
    id = Column(Integer, primary_key=True)
    payment_id = Column(String(64), unique=True, index=True, comment="全局支付ID")

    # 平台信息
    platform = Column(String(20), nullable=False, comment="支付平台")
    platform_order_id = Column(String(64), nullable=False, comment="平台订单ID")
    platform_transaction_id = Column(String(64), nullable=True, comment="平台交易ID")

    # 支付信息
    amount = Column(Decimal(10, 2), nullable=False, comment="支付金额")
    currency = Column(String(3), default="CNY", comment="货币类型")
    title = Column(String(128), nullable=False, comment="支付标题")
    description = Column(Text, nullable=True, comment="支付描述")

    # 用户信息
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False, comment="用户ID")
    user_ip = Column(String(45), nullable=True, comment="用户IP")

    # 状态管理
    unified_status = Column(String(20), nullable=False, default="pending", comment="统一状态")
    platform_status = Column(String(50), nullable=True, comment="平台原始状态")
    status_updated_at = Column(DateTime, nullable=True, comment="状态更新时间")

    # 时间字段
    expires_at = Column(DateTime, nullable=True, comment="过期时间")
    paid_at = Column(DateTime, nullable=True, comment="支付完成时间")

    # 扩展信息
    metadata = Column(JSON, nullable=True, comment="扩展元数据")

    # 审计字段
    created_at = Column(DateTime, default=datetime.utcnow, comment="创建时间")
    updated_at = Column(DateTime, onupdate=datetime.utcnow, comment="更新时间")
    deleted_at = Column(DateTime, nullable=True, comment="软删除时间")

    def update_status(self, new_status: UnifiedPaymentStatus, platform_status: str = None):
        """更新支付状态"""
        if not self.can_transition_to(new_status):
            raise ValueError(f"Cannot transition from {self.unified_status} to {new_status}")

        self.unified_status = new_status.value
        self.platform_status = platform_status
        self.status_updated_at = datetime.utcnow()

        if new_status == UnifiedPaymentStatus.SUCCESS:
            self.paid_at = datetime.utcnow()

    def can_transition_to(self, target_status: UnifiedPaymentStatus) -> bool:
        """检查是否可以转换到目标状态"""
        current = UnifiedPaymentStatus(self.unified_status)
        return target_status in VALID_STATUS_TRANSITIONS.get(current, [])

    def is_final_status(self) -> bool:
        """检查是否为终态"""
        current = UnifiedPaymentStatus(self.unified_status)
        return len(VALID_STATUS_TRANSITIONS.get(current, [])) == 0

    def soft_delete(self):
        """软删除"""
        self.deleted_at = datetime.utcnow()
```

#### 2.2.2 PaymentLog 模型
```python
class PaymentLog(BaseModel):
    """支付操作日志模型"""
    __tablename__ = "payment_logs"

    id = Column(Integer, primary_key=True)
    payment_id = Column(String(64), ForeignKey("unified_payments.payment_id"), comment="支付ID")

    # 操作信息
    action_type = Column(String(20), nullable=False, comment="操作类型")  # CREATE/UPDATE/CALLBACK/REFUND/QUERY
    platform = Column(String(20), nullable=False, comment="支付平台")

    # 状态变更
    status_from = Column(String(20), nullable=True, comment="原状态")
    status_to = Column(String(20), nullable=True, comment="新状态")

    # 请求响应数据 (脱敏处理)
    request_data = Column(JSON, nullable=True, comment="请求数据")
    response_data = Column(JSON, nullable=True, comment="响应数据")

    # 操作上下文
    user_id = Column(Integer, nullable=True, comment="操作用户ID")
    ip_address = Column(String(45), nullable=True, comment="操作IP")
    user_agent = Column(Text, nullable=True, comment="用户代理")

    # 结果信息
    success = Column(Boolean, nullable=False, default=True, comment="操作是否成功")
    error_code = Column(String(50), nullable=True, comment="错误码")
    error_message = Column(Text, nullable=True, comment="错误信息")

    # 审计字段
    created_at = Column(DateTime, default=datetime.utcnow, comment="创建时间")

    @classmethod
    def create_log(cls, payment_id: str, action_type: str, platform: str, **kwargs):
        """创建日志记录"""
        return cls(
            payment_id=payment_id,
            action_type=action_type,
            platform=platform,
            **kwargs
        )
```

#### 2.2.3 RefundRecord 模型
```python
class RefundRecord(BaseModel):
    """退款记录模型"""
    __tablename__ = "refund_records"

    id = Column(Integer, primary_key=True)
    refund_id = Column(String(64), unique=True, index=True, comment="退款唯一ID")
    payment_id = Column(String(64), ForeignKey("unified_payments.payment_id"), comment="关联支付ID")

    # 平台信息
    platform = Column(String(20), nullable=False, comment="支付平台")
    platform_refund_id = Column(String(64), nullable=True, comment="平台退款ID")

    # 退款信息
    refund_amount = Column(Decimal(10, 2), nullable=False, comment="退款金额")
    refund_reason = Column(String(256), nullable=True, comment="退款原因")
    refund_type = Column(String(20), nullable=False, comment="退款类型")  # FULL/PARTIAL

    # 状态管理
    status = Column(String(20), nullable=False, default="pending", comment="退款状态")
    platform_status = Column(String(50), nullable=True, comment="平台退款状态")

    # 操作信息
    operator_id = Column(Integer, nullable=True, comment="操作员ID")
    operator_type = Column(String(20), nullable=False, comment="操作类型")  # SYSTEM/MANUAL

    # 时间字段
    processed_at = Column(DateTime, nullable=True, comment="处理完成时间")
    created_at = Column(DateTime, default=datetime.utcnow, comment="创建时间")
    updated_at = Column(DateTime, onupdate=datetime.utcnow, comment="更新时间")

    def update_status(self, new_status: str, platform_status: str = None):
        """更新退款状态"""
        self.status = new_status
        self.platform_status = platform_status

        if new_status in ["success", "failed"]:
            self.processed_at = datetime.utcnow()
```

### 2.3 统一API接口设计

#### 2.3.1 核心API端点

**1. 查询支付状态**
```python
@router.get("/api/payments/status/{payment_id}")
async def get_payment_status(
    payment_id: str,
    include_logs: bool = False,
    db: Session = Depends(get_db)
) -> PaymentStatusResponse:
    """查询支付状态

    Args:
        payment_id: 支付ID
        include_logs: 是否包含操作日志

    Returns:
        PaymentStatusResponse: 支付状态信息
    """

# 响应模型
class PaymentStatusResponse(BaseModel):
    payment_id: str
    platform: str
    unified_status: str
    platform_status: Optional[str]
    amount: Decimal
    currency: str
    title: str
    user_id: int
    created_at: datetime
    paid_at: Optional[datetime]
    expires_at: Optional[datetime]
    logs: Optional[List[PaymentLogInfo]] = None
```

**2. 验证支付状态**
```python
@router.post("/api/payments/verify")
async def verify_payment_status(
    request: PaymentVerifyRequest,
    db: Session = Depends(get_db)
) -> PaymentVerifyResponse:
    """验证支付状态

    支持单个验证和批量验证
    """

# 请求模型
class PaymentVerifyRequest(BaseModel):
    payment_ids: List[str] = Field(..., max_items=100, description="支付ID列表")
    force_sync: bool = Field(False, description="是否强制同步")

# 响应模型
class PaymentVerifyResponse(BaseModel):
    total_count: int
    success_count: int
    failed_count: int
    results: List[PaymentVerifyResult]

class PaymentVerifyResult(BaseModel):
    payment_id: str
    verified: bool
    unified_status: str
    platform_status: Optional[str]
    error_message: Optional[str]
```

**3. 申请退款**
```python
@router.post("/api/payments/refund")
async def create_refund(
    request: RefundCreateRequest,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
) -> RefundCreateResponse:
    """申请退款"""

# 请求模型
class RefundCreateRequest(BaseModel):
    payment_id: str
    refund_amount: Optional[Decimal] = None  # None表示全额退款
    refund_reason: str = Field(..., max_length=256)
    notify_user: bool = Field(True, description="是否通知用户")

# 响应模型
class RefundCreateResponse(BaseModel):
    refund_id: str
    payment_id: str
    refund_amount: Decimal
    status: str
    estimated_arrival: Optional[str]  # 预计到账时间
```

**4. 获取支付日志**
```python
@router.get("/api/payments/logs/{payment_id}")
async def get_payment_logs(
    payment_id: str,
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    action_type: Optional[str] = None,
    db: Session = Depends(get_db)
) -> PaymentLogsResponse:
    """获取支付日志"""

class PaymentLogsResponse(BaseModel):
    payment_id: str
    total_count: int
    page: int
    page_size: int
    logs: List[PaymentLogInfo]
```

**5. 同步支付状态**
```python
@router.post("/api/payments/sync-status")
async def sync_payment_status(
    request: StatusSyncRequest,
    db: Session = Depends(get_db)
) -> StatusSyncResponse:
    """主动同步支付状态"""

class StatusSyncRequest(BaseModel):
    payment_ids: Optional[List[str]] = None  # None表示同步所有待处理支付
    platform: Optional[str] = None  # 指定平台
    max_age_hours: int = Field(24, description="同步最近N小时内的支付")

class StatusSyncResponse(BaseModel):
    sync_count: int
    success_count: int
    failed_count: int
    results: List[StatusSyncResult]
```

### 2.4 状态同步机制

#### 2.4.1 PaymentStatusSyncService
```python
class PaymentStatusSyncService:
    """支付状态同步服务"""

    def __init__(self, db: Session):
        self.db = db
        self.platform_adapters = {
            PaymentPlatform.ALIPAY: AlipayAdapter(),
            PaymentPlatform.WECHAT: WechatAdapter(),
            PaymentPlatform.UNIONPAY: UnionPayAdapter()
        }

    async def sync_payment_status(self, payment_id: str) -> SyncResult:
        """同步单个支付状态"""
        payment = await self.get_payment(payment_id)
        if not payment:
            return SyncResult(success=False, error="Payment not found")

        platform = PaymentPlatform(payment.platform)
        adapter = self.platform_adapters[platform]

        try:
            # 查询平台最新状态
            platform_status = await adapter.query_payment_status(
                payment.platform_order_id
            )

            # 映射到统一状态
            unified_status = self.map_platform_status(platform, platform_status)

            # 更新本地状态
            if unified_status != UnifiedPaymentStatus(payment.unified_status):
                await self.update_payment_status(
                    payment, unified_status, platform_status
                )

            return SyncResult(success=True, status_changed=True)

        except Exception as e:
            logger.error(f"Failed to sync payment {payment_id}: {e}")
            return SyncResult(success=False, error=str(e))

    async def batch_sync_pending_payments(self) -> List[SyncResult]:
        """批量同步待处理支付"""
        pending_payments = await self.get_pending_payments()

        # 并发同步，控制并发数
        semaphore = asyncio.Semaphore(10)

        async def sync_with_semaphore(payment_id):
            async with semaphore:
                return await self.sync_payment_status(payment_id)

        tasks = [sync_with_semaphore(p.payment_id) for p in pending_payments]
        results = await asyncio.gather(*tasks, return_exceptions=True)

        return [r if isinstance(r, SyncResult) else
                SyncResult(success=False, error=str(r)) for r in results]

    async def handle_platform_callback(
        self, platform: str, callback_data: dict
    ) -> bool:
        """处理平台回调"""
        try:
            platform_enum = PaymentPlatform(platform)
            adapter = self.platform_adapters[platform_enum]

            # 验证回调签名
            if not adapter.verify_callback_signature(callback_data):
                logger.warning(f"Invalid callback signature from {platform}")
                return False

            # 提取支付信息
            platform_order_id = adapter.extract_order_id(callback_data)
            platform_status = adapter.extract_status(callback_data)

            # 查找本地支付记录
            payment = await self.get_payment_by_platform_order_id(
                platform, platform_order_id
            )

            if not payment:
                logger.warning(f"Payment not found for {platform}:{platform_order_id}")
                return False

            # 映射并更新状态
            unified_status = self.map_platform_status(platform_enum, platform_status)
            await self.update_payment_status(payment, unified_status, platform_status)

            # 记录回调日志
            await self.create_callback_log(payment.payment_id, platform, callback_data)

            return True

        except Exception as e:
            logger.error(f"Failed to handle callback from {platform}: {e}")
            return False

    def map_platform_status(
        self, platform: PaymentPlatform, platform_status: str
    ) -> UnifiedPaymentStatus:
        """映射平台状态到统一状态"""
        mapping = PLATFORM_STATUS_MAPPING.get(platform, {})
        return mapping.get(platform_status, UnifiedPaymentStatus.UNKNOWN)
```

#### 2.4.2 定时同步任务
```python
# 使用Celery定时任务
@celery_app.task
def sync_pending_payments_task():
    """定时同步待处理支付状态"""
    with SessionLocal() as db:
        sync_service = PaymentStatusSyncService(db)
        results = asyncio.run(sync_service.batch_sync_pending_payments())

        # 统计结果
        success_count = sum(1 for r in results if r.success)
        failed_count = len(results) - success_count

        logger.info(f"Payment sync completed: {success_count} success, {failed_count} failed")

# 配置定时任务 (每5分钟执行一次)
celery_app.conf.beat_schedule = {
    'sync-payments': {
        'task': 'sync_pending_payments_task',
        'schedule': crontab(minute='*/5'),
    },
}
```

### 2.5 通知机制

#### 2.5.1 事件系统
```python
class PaymentEvent(BaseModel):
    """支付事件模型"""
    __tablename__ = "payment_events"

    id = Column(Integer, primary_key=True)
    event_id = Column(String(64), unique=True, index=True)
    payment_id = Column(String(64), nullable=False)
    event_type = Column(String(50), nullable=False)  # status_changed/refund_created
    event_data = Column(JSON, nullable=True)
    processed = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)

class PaymentEventService:
    """支付事件服务"""

    async def publish_event(self, event_type: str, payment_id: str, data: dict):
        """发布支付事件"""
        event = PaymentEvent(
            event_id=str(uuid.uuid4()),
            payment_id=payment_id,
            event_type=event_type,
            event_data=data
        )

        await self.save_event(event)
        await self.notify_subscribers(event)

    async def notify_subscribers(self, event: PaymentEvent):
        """通知订阅者"""
        subscribers = await self.get_event_subscribers(event.event_type)

        for subscriber in subscribers:
            try:
                await subscriber.handle_event(event)
            except Exception as e:
                logger.error(f"Failed to notify subscriber {subscriber}: {e}")
```

#### 2.5.2 Webhook通知
```python
class WebhookNotificationService:
    """Webhook通知服务"""

    async def send_payment_notification(
        self, payment: UnifiedPayment, event_type: str
    ):
        """发送支付通知"""
        webhook_urls = await self.get_webhook_urls(payment.user_id)

        notification_data = {
            "event_type": event_type,
            "payment_id": payment.payment_id,
            "status": payment.unified_status,
            "amount": str(payment.amount),
            "timestamp": datetime.utcnow().isoformat()
        }

        for url in webhook_urls:
            await self.send_webhook(url, notification_data)

    async def send_webhook(self, url: str, data: dict, max_retries: int = 3):
        """发送Webhook请求"""
        for attempt in range(max_retries):
            try:
                async with httpx.AsyncClient() as client:
                    response = await client.post(
                        url,
                        json=data,
                        timeout=10.0,
                        headers={"Content-Type": "application/json"}
                    )

                if response.status_code == 200:
                    return True

            except Exception as e:
                logger.warning(f"Webhook attempt {attempt + 1} failed: {e}")

            if attempt < max_retries - 1:
                await asyncio.sleep(2 ** attempt)  # 指数退避

        return False
```

### 2.6 平台适配器设计

#### 2.6.1 统一适配器接口
```python
from abc import ABC, abstractmethod

class PaymentPlatformAdapter(ABC):
    """支付平台适配器基类"""

    @abstractmethod
    async def query_payment_status(self, platform_order_id: str) -> str:
        """查询支付状态"""
        pass

    @abstractmethod
    def verify_callback_signature(self, callback_data: dict) -> bool:
        """验证回调签名"""
        pass

    @abstractmethod
    def extract_order_id(self, callback_data: dict) -> str:
        """提取订单ID"""
        pass

    @abstractmethod
    def extract_status(self, callback_data: dict) -> str:
        """提取支付状态"""
        pass

    @abstractmethod
    async def create_refund(self, refund_request: RefundRequest) -> RefundResult:
        """创建退款"""
        pass

class AlipayAdapter(PaymentPlatformAdapter):
    """支付宝适配器"""

    def __init__(self, alipay_service: AlipayService):
        self.alipay_service = alipay_service

    async def query_payment_status(self, platform_order_id: str) -> str:
        """查询支付宝支付状态"""
        return await self.alipay_service.query_payment_status(platform_order_id)

    def verify_callback_signature(self, callback_data: dict) -> bool:
        """验证支付宝回调签名"""
        return self.alipay_service.verify_callback_signature(callback_data)

    # ... 其他方法实现

class WechatAdapter(PaymentPlatformAdapter):
    """微信支付适配器"""

    def __init__(self, wechat_service: WechatPayService):
        self.wechat_service = wechat_service

    # ... 实现抽象方法

class UnionPayAdapter(PaymentPlatformAdapter):
    """银联支付适配器"""

    def __init__(self, unionpay_service: UnionPayService):
        self.unionpay_service = unionpay_service

    # ... 实现抽象方法
```

## 3. 集成设计

### 3.1 与现有系统集成策略

#### 3.1.1 非侵入式集成
- **保持独立**: 现有支付宝、微信、银联服务继续独立运行
- **适配器模式**: 通过适配器集成现有服务，不修改原有代码
- **数据映射**: 建立平台数据到统一模型的映射关系
- **渐进迁移**: 支持新老API并存，逐步迁移

#### 3.1.2 数据同步策略
```python
class PaymentDataMigrationService:
    """支付数据迁移服务"""

    async def migrate_existing_payments(self):
        """迁移现有支付数据到统一模型"""
        # 迁移支付宝订单
        await self.migrate_alipay_orders()

        # 迁移微信支付订单
        await self.migrate_wechat_orders()

        # 迁移银联支付订单
        await self.migrate_unionpay_orders()

    async def migrate_alipay_orders(self):
        """迁移支付宝订单"""
        alipay_orders = await self.get_all_alipay_orders()

        for order in alipay_orders:
            unified_payment = UnifiedPayment(
                payment_id=f"ALI_{order.out_trade_no}",
                platform=PaymentPlatform.ALIPAY.value,
                platform_order_id=order.out_trade_no,
                platform_transaction_id=order.trade_no,
                amount=order.total_amount,
                title=order.subject,
                user_id=order.user_id,
                unified_status=self.map_alipay_status(order.trade_status),
                platform_status=order.trade_status,
                created_at=order.created_at,
                paid_at=order.gmt_payment
            )

            await self.save_unified_payment(unified_payment)
```

#### 3.1.3 API兼容性设计
```python
# 保持现有API不变，新增统一API
# 现有API: /api/payments/alipay/create
# 统一API: /api/payments/create (支持platform参数)

@router.post("/api/payments/create")
async def create_unified_payment(
    request: UnifiedPaymentCreateRequest,
    db: Session = Depends(get_db)
) -> UnifiedPaymentCreateResponse:
    """统一支付创建接口"""

    # 根据平台路由到对应服务
    if request.platform == "alipay":
        return await create_alipay_payment(request)
    elif request.platform == "wechat":
        return await create_wechat_payment(request)
    elif request.platform == "unionpay":
        return await create_unionpay_payment(request)
```

### 3.2 架构分层设计

#### 3.2.1 分层架构图
```
┌─────────────────────────────────────────────────────────┐
│                    API Gateway层                         │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────┐ │
│  │ 统一支付API     │ │ 支付状态API     │ │ 退款API     │ │
│  └─────────────────┘ └─────────────────┘ └─────────────┘ │
└─────────────────────────────────────────────────────────┘
                           │
┌─────────────────────────────────────────────────────────┐
│                    业务服务层                           │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────┐ │
│  │PaymentStatusSvc │ │PaymentSyncSvc   │ │RefundSvc    │ │
│  └─────────────────┘ └─────────────────┘ └─────────────┘ │
└─────────────────────────────────────────────────────────┘
                           │
┌─────────────────────────────────────────────────────────┐
│                    平台适配层                           │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────┐ │
│  │AlipayAdapter    │ │WechatAdapter    │ │UnionAdapter │ │
│  └─────────────────┘ └─────────────────┘ └─────────────┘ │
└─────────────────────────────────────────────────────────┘
                           │
┌─────────────────────────────────────────────────────────┐
│                    平台服务层                           │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────┐ │
│  │AlipayService    │ │WechatPayService │ │UnionPaySvc  │ │
│  └─────────────────┘ └─────────────────┘ └─────────────┘ │
└─────────────────────────────────────────────────────────┘
                           │
┌─────────────────────────────────────────────────────────┐
│                     数据访问层                          │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────┐ │
│  │UnifiedPayment   │ │PaymentLog       │ │RefundRecord │ │
│  └─────────────────┘ └─────────────────┘ └─────────────┘ │
└─────────────────────────────────────────────────────────┘
```

#### 3.2.2 依赖注入配置
```python
# 依赖注入配置
from dependency_injector import containers, providers

class PaymentContainer(containers.DeclarativeContainer):
    """支付系统依赖注入容器"""

    # 数据访问层
    db_session = providers.Resource(get_db_session)

    # 平台服务层
    alipay_service = providers.Factory(AlipayService, db=db_session)
    wechat_service = providers.Factory(WechatPayService, db=db_session)
    unionpay_service = providers.Factory(UnionPayService, db=db_session)

    # 适配器层
    alipay_adapter = providers.Factory(AlipayAdapter, alipay_service=alipay_service)
    wechat_adapter = providers.Factory(WechatAdapter, wechat_service=wechat_service)
    unionpay_adapter = providers.Factory(UnionPayAdapter, unionpay_service=unionpay_service)

    # 业务服务层
    payment_status_service = providers.Factory(
        PaymentStatusService,
        db=db_session,
        alipay_adapter=alipay_adapter,
        wechat_adapter=wechat_adapter,
        unionpay_adapter=unionpay_adapter
    )

    payment_sync_service = providers.Factory(PaymentStatusSyncService, db=db_session)
    refund_service = providers.Factory(RefundService, db=db_session)
```

## 4. 实现路径

### 4.1 分阶段实施计划

#### Phase 1: 基础架构 (1工作日)
**目标**: 建立统一支付状态管理的基础设施

**任务清单**:
- [ ] 创建统一支付数据模型 (UnifiedPayment, PaymentLog, RefundRecord)
- [ ] 编写数据库迁移脚本
- [ ] 实现状态枚举和状态机验证
- [ ] 建立基础的依赖注入配置
- [ ] 编写数据模型单元测试

**验收标准**:
- [ ] 数据模型创建成功，支持基础CRUD操作
- [ ] 状态机转换规则正确实现
- [ ] 单元测试覆盖率>90%

#### Phase 2: 状态管理核心 (1工作日)
**目标**: 实现统一支付状态查询、验证和日志功能

**任务清单**:
- [ ] 实现PaymentStatusService核心服务
- [ ] 开发统一状态查询API接口
- [ ] 实现支付状态验证功能
- [ ] 建立支付操作日志记录机制
- [ ] 创建平台适配器基础框架

**验收标准**:
- [ ] 支持查询任意平台的支付状态
- [ ] 状态验证功能正常工作
- [ ] 操作日志完整记录
- [ ] API响应时间<500ms

#### Phase 3: 退款和异常处理 (0.5工作日)
**目标**: 实现统一退款管理和异常处理机制

**任务清单**:
- [ ] 实现RefundService退款服务
- [ ] 开发统一退款API接口
- [ ] 建立异常处理和重试机制
- [ ] 实现退款状态跟踪
- [ ] 完善错误码和错误处理

**验收标准**:
- [ ] 支持全额和部分退款
- [ ] 退款状态正确跟踪
- [ ] 异常情况正确处理
- [ ] 防重复退款机制有效

#### Phase 4: 同步和通知 (0.5工作日)
**目标**: 实现状态同步和事件通知机制

**任务清单**:
- [ ] 实现PaymentStatusSyncService同步服务
- [ ] 建立定时同步任务
- [ ] 开发事件通知系统
- [ ] 实现Webhook通知功能
- [ ] 完善监控和告警

**验收标准**:
- [ ] 定时同步任务正常运行
- [ ] 回调处理正确工作
- [ ] 事件通知及时触发
- [ ] 监控指标正确统计

### 4.2 关键里程碑

| 里程碑 | 时间点 | 验收标准 |
|--------|--------|----------|
| M1 - 基础架构完成 | Day 1 | 数据模型和基础服务就绪 |
| M2 - 状态管理可用 | Day 2 | 统一状态查询和验证功能可用 |
| M3 - 退款功能完成 | Day 2.5 | 统一退款功能完整实现 |
| M4 - 系统集成完成 | Day 3 | 所有功能集成并通过测试 |

### 4.3 质量保证措施

#### 4.3.1 测试策略
```python
# 单元测试示例
class TestPaymentStatusService:
    """支付状态服务测试"""

    async def test_query_payment_status(self):
        """测试支付状态查询"""
        # 准备测试数据
        payment = create_test_payment()

        # 执行查询
        result = await self.payment_service.get_payment_status(payment.payment_id)

        # 验证结果
        assert result.payment_id == payment.payment_id
        assert result.unified_status == payment.unified_status

    async def test_status_transition_validation(self):
        """测试状态转换验证"""
        payment = create_test_payment(status="pending")

        # 合法转换
        result = await self.payment_service.update_status(
            payment.payment_id, "success"
        )
        assert result.success

        # 非法转换
        with pytest.raises(ValueError):
            await self.payment_service.update_status(
                payment.payment_id, "pending"
            )

    async def test_platform_status_mapping(self):
        """测试平台状态映射"""
        # 测试支付宝状态映射
        unified_status = map_platform_status(
            PaymentPlatform.ALIPAY, "TRADE_SUCCESS"
        )
        assert unified_status == UnifiedPaymentStatus.SUCCESS

        # 测试微信状态映射
        unified_status = map_platform_status(
            PaymentPlatform.WECHAT, "SUCCESS"
        )
        assert unified_status == UnifiedPaymentStatus.SUCCESS
```

#### 4.3.2 集成测试
```python
class TestPaymentIntegration:
    """支付系统集成测试"""

    async def test_end_to_end_payment_flow(self):
        """端到端支付流程测试"""
        # 1. 创建支付
        payment = await self.create_test_payment()

        # 2. 模拟平台回调
        callback_data = create_test_callback(payment.platform_order_id)
        result = await self.sync_service.handle_platform_callback(
            payment.platform, callback_data
        )
        assert result

        # 3. 验证状态更新
        updated_payment = await self.get_payment(payment.payment_id)
        assert updated_payment.unified_status == "success"

        # 4. 创建退款
        refund = await self.create_refund(payment.payment_id)
        assert refund.status == "pending"

    async def test_concurrent_status_updates(self):
        """并发状态更新测试"""
        payment = await self.create_test_payment()

        # 并发更新状态
        tasks = [
            self.update_payment_status(payment.payment_id, "processing"),
            self.handle_callback(payment.platform_order_id, "success")
        ]

        results = await asyncio.gather(*tasks, return_exceptions=True)

        # 验证最终状态一致性
        final_payment = await self.get_payment(payment.payment_id)
        assert final_payment.unified_status in ["processing", "success"]
```

#### 4.3.3 性能测试
```python
class TestPaymentPerformance:
    """支付系统性能测试"""

    async def test_batch_status_query_performance(self):
        """批量状态查询性能测试"""
        # 创建1000个测试支付
        payment_ids = await self.create_test_payments(1000)

        # 测试批量查询性能
        start_time = time.time()
        results = await self.payment_service.batch_get_payment_status(payment_ids)
        end_time = time.time()

        # 验证性能要求
        assert end_time - start_time < 5.0  # 5秒内完成
        assert len(results) == 1000
        assert all(r.success for r in results)

    async def test_concurrent_callback_handling(self):
        """并发回调处理性能测试"""
        # 模拟100个并发回调
        tasks = []
        for i in range(100):
            callback_data = create_test_callback(f"order_{i}")
            task = self.sync_service.handle_platform_callback("alipay", callback_data)
            tasks.append(task)

        start_time = time.time()
        results = await asyncio.gather(*tasks)
        end_time = time.time()

        # 验证性能和正确性
        assert end_time - start_time < 10.0  # 10秒内完成
        assert all(results)  # 所有回调处理成功
```

## 5. 风险评估与缓解

### 5.1 技术风险

#### 5.1.1 状态一致性风险
**风险描述**: 多平台状态同步可能出现数据不一致
**影响程度**: 高
**缓解措施**:
- 实现分布式锁防止并发修改
- 建立数据一致性检查和修复机制
- 使用事务确保原子性操作
- 定期对账和状态校验

#### 5.1.2 性能风险
**风险描述**: 统一查询可能成为性能瓶颈
**影响程度**: 中等
**缓解措施**:
- 实现Redis缓存热点数据
- 使用数据库读写分离
- 实现异步处理和批量操作
- 建立性能监控和告警

#### 5.1.3 平台兼容性风险
**风险描述**: 各支付平台API变更影响适配器
**影响程度**: 中等
**缓解措施**:
- 建立API版本管理机制
- 实现降级处理和容错机制
- 定期监控平台API状态
- 建立快速响应和修复流程

### 5.2 业务风险

#### 5.2.1 数据迁移风险
**风险描述**: 现有数据迁移可能丢失或错误
**影响程度**: 高
**缓解措施**:
- 实现完整的数据备份机制
- 分批次渐进式迁移
- 建立数据验证和回滚机制
- 充分的迁移测试

#### 5.2.2 服务中断风险
**风险描述**: 新系统上线可能影响现有服务
**影响程度**: 高
**缓解措施**:
- 采用蓝绿部署策略
- 实现平滑切换机制
- 建立快速回滚方案
- 充分的灰度测试

### 5.3 进度风险

#### 5.3.1 技术复杂度风险
**风险描述**: 多平台集成复杂度超出预期
**影响程度**: 中等
**缓解措施**:
- 分阶段实施降低复杂度
- 预留额外缓冲时间
- 寻求技术专家支持
- 准备降级方案

## 6. 监控和维护

### 6.1 监控指标设计

#### 6.1.1 业务指标
```python
class PaymentMetrics:
    """支付业务指标"""

    @staticmethod
    async def get_payment_success_rate(
        platform: str = None,
        hours: int = 24
    ) -> Dict[str, float]:
        """获取支付成功率"""
        query = """
        SELECT
            platform,
            COUNT(*) as total,
            COUNT(CASE WHEN unified_status = 'success' THEN 1 END) as success
        FROM unified_payments
        WHERE created_at >= NOW() - INTERVAL %s HOUR
        """

        if platform:
            query += " AND platform = %s"

        query += " GROUP BY platform"

        # 执行查询并计算成功率
        # ...

    @staticmethod
    async def get_average_payment_duration() -> Dict[str, float]:
        """获取平均支付时长"""
        query = """
        SELECT
            platform,
            AVG(TIMESTAMPDIFF(SECOND, created_at, paid_at)) as avg_duration
        FROM unified_payments
        WHERE unified_status = 'success'
            AND paid_at IS NOT NULL
            AND created_at >= NOW() - INTERVAL 24 HOUR
        GROUP BY platform
        """

        # 执行查询
        # ...

    @staticmethod
    async def get_refund_statistics() -> Dict[str, Any]:
        """获取退款统计"""
        # 退款率、退款成功率、平均退款时长等
        # ...
```

#### 6.1.2 技术指标
```python
class TechnicalMetrics:
    """技术监控指标"""

    @staticmethod
    async def get_api_performance() -> Dict[str, Dict[str, float]]:
        """获取API性能指标"""
        return {
            "payment_status_query": {
                "avg_response_time": 0.15,  # 秒
                "95th_percentile": 0.3,
                "error_rate": 0.001
            },
            "payment_verify": {
                "avg_response_time": 0.25,
                "95th_percentile": 0.5,
                "error_rate": 0.002
            }
        }

    @staticmethod
    async def get_sync_service_health() -> Dict[str, Any]:
        """获取同步服务健康状态"""
        return {
            "last_sync_time": datetime.utcnow(),
            "pending_sync_count": 0,
            "sync_success_rate": 0.99,
            "sync_lag_seconds": 30
        }

    @staticmethod
    async def get_database_performance() -> Dict[str, float]:
        """获取数据库性能指标"""
        return {
            "avg_query_time": 0.05,
            "connection_pool_usage": 0.6,
            "slow_query_count": 0
        }
```

#### 6.1.3 告警规则
```python
ALERT_RULES = {
    # 业务告警
    "payment_success_rate_low": {
        "condition": "success_rate < 0.95",
        "duration": "5m",
        "severity": "warning"
    },
    "payment_timeout_high": {
        "condition": "timeout_rate > 0.05",
        "duration": "3m",
        "severity": "critical"
    },

    # 技术告警
    "api_response_time_high": {
        "condition": "avg_response_time > 1.0",
        "duration": "2m",
        "severity": "warning"
    },
    "sync_service_down": {
        "condition": "sync_lag_seconds > 300",
        "duration": "1m",
        "severity": "critical"
    },

    # 系统告警
    "database_connection_high": {
        "condition": "connection_pool_usage > 0.9",
        "duration": "5m",
        "severity": "warning"
    }
}
```

### 6.2 运维工具

#### 6.2.1 数据修复工具
```python
class PaymentDataRepairTool:
    """支付数据修复工具"""

    async def repair_inconsistent_status(self, payment_id: str) -> bool:
        """修复状态不一致的支付"""
        payment = await self.get_payment(payment_id)

        # 查询平台真实状态
        platform_status = await self.query_platform_status(
            payment.platform, payment.platform_order_id
        )

        # 修正本地状态
        correct_status = self.map_platform_status(
            payment.platform, platform_status
        )

        if correct_status != payment.unified_status:
            await self.update_payment_status(payment_id, correct_status)
            return True

        return False

    async def batch_repair_payments(
        self, status: str = None, hours: int = 24
    ) -> List[str]:
        """批量修复支付数据"""
        problematic_payments = await self.find_problematic_payments(status, hours)

        repaired = []
        for payment_id in problematic_payments:
            if await self.repair_inconsistent_status(payment_id):
                repaired.append(payment_id)

        return repaired
```

#### 6.2.2 对账工具
```python
class PaymentReconciliationTool:
    """支付对账工具"""

    async def reconcile_daily_payments(self, date: str) -> ReconciliationReport:
        """日对账"""
        # 获取本地支付记录
        local_payments = await self.get_local_payments(date)

        # 获取各平台对账文件
        platform_records = {}
        for platform in [PaymentPlatform.ALIPAY, PaymentPlatform.WECHAT, PaymentPlatform.UNIONPAY]:
            platform_records[platform] = await self.get_platform_records(platform, date)

        # 执行对账
        differences = []
        for payment in local_payments:
            platform_record = platform_records[payment.platform].get(payment.platform_order_id)

            if not platform_record:
                differences.append({
                    "type": "missing_platform_record",
                    "payment_id": payment.payment_id
                })
            elif platform_record.status != payment.platform_status:
                differences.append({
                    "type": "status_mismatch",
                    "payment_id": payment.payment_id,
                    "local_status": payment.platform_status,
                    "platform_status": platform_record.status
                })

        return ReconciliationReport(
            date=date,
            total_local_payments=len(local_payments),
            total_platform_records=sum(len(records) for records in platform_records.values()),
            differences=differences
        )
```

## 7. 总结

### 7.1 技术方案优势

1. **统一抽象**: 通过统一的支付状态模型和API，简化了多平台支付管理
2. **非侵入式**: 采用适配器模式，不破坏现有支付系统的独立性
3. **高可用性**: 实现了完善的状态同步、异常处理和监控机制
4. **可扩展性**: 分层架构设计，便于接入新的支付平台
5. **数据一致性**: 通过状态机和事务确保数据一致性
6. **监控完善**: 建立了全面的业务和技术监控体系

### 7.2 实施建议

1. **分阶段实施**: 按照4个阶段逐步实施，降低风险
2. **充分测试**: 重点关注数据一致性和并发场景测试
3. **监控先行**: 建立监控体系，确保系统健康可观测
4. **文档同步**: 开发过程中同步更新技术文档和操作手册

### 7.3 预期效果

- **开发效率提升**: 统一API减少重复开发工作
- **运维效率提升**: 集中化的监控和管理界面
- **业务价值提升**: 完整的支付数据分析和洞察能力
- **系统稳定性提升**: 完善的异常处理和恢复机制

### 7.4 后续规划

1. **功能增强**: 支持更多支付平台和支付方式
2. **智能化**: 基于数据分析的智能路由和风控
3. **国际化**: 支持海外支付平台和多币种
4. **开放平台**: 为第三方开发者提供支付API

---

**文档状态**: ✅ 分析完成
**技术方案**: ✅ 设计完成
**实施计划**: ✅ 规划完成
**下一步行动**: 等待技术方案评审，准备启动开发

**总估时**: 3工作日 (符合Task 037要求)
**核心交付物**: 统一支付状态管理系统，支持查询、验证、退款、同步、日志和通知功能