# CalorWise 方案C改造总纲（iOS + 后台 + 接口Schema + 废弃接口 + 数据库）

Date: 2026-02-10

## Project context
- App(iOS): `/Users/lennox/CalorWiseProject/CalorWise`
- Backend(Flask): `/Users/lennox/code/projects/CalorWise`
- 目标：从“设备 token + 客户端判权”迁移到“后端主导账号与订阅权益”。

## 1) iOS 如何改造

### 1.1 认证与会话
- 新增 `AuthRepository`，引入 `accessToken + refreshToken`（Keychain 存储）。
- 保留游客进入能力，但游客会话由后端签发（不再只依赖本地 `get_token` 缓存）。
- 登录状态来源改为“后端会话有效性”，替换本地布尔 `isLogin` 作为真值。

### 1.2 订阅链路
- StoreKit 仍由客户端发起购买/恢复。
- 购买回调后立即调用后端 `POST /v1/subscription/client-event`。
- 页面权益展示统一调用 `GET /v1/subscription/entitlement`。
- 删除本地最终判权：本地 7 天放行、客户端 shared secret 最终裁决。

### 1.3 网络与安全
- 全量切 HTTPS。
- 收紧 ATS（移除全局 `NSAllowsArbitraryLoads = true`）。
- 旧接口兼容期保留，统一封装 Header：`Authorization: Bearer <access_token>`。

### 1.4 数据并档
- 游客登录成功后调用 `POST /v1/account/merge-guest`。
- 并档完成后刷新本地 cache（daily log / settings / favorites / weight）。

## 2) 后台如何改造

### 2.1 服务分层
- `Auth/Account`：登录、会话、并档、删号。
- `Subscription`：交易接收、通知消费、权益聚合、补偿拉取。
- `Gateway/Middleware`：鉴权、限流、审计、幂等。

### 2.2 鉴权机制升级
- 旧：请求体 `token` + DES 解密。
- 新：Bearer session token + refresh token 轮换。
- 兼容期：中间件支持双栈（旧 token 与新 bearer 并存），按版本灰度切流。

### 2.3 订阅服务能力
- 接 App Store Server Notifications V2。
- 定时 reconcile（主动拉取）补偿通知延迟/丢失。
- 构建 `originalTransactionId` 维度状态机：active / grace / retry / expired / revoked。

## 3) 接口定义与 schema（建议 v1）

### 3.1 Auth / Account

#### `POST /v1/auth/guest/session`
Request
```json
{
  "deviceId": "ios-device-uuid",
  "platform": "ios",
  "appVersion": "1.2.0"
}
```
Response
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "guestUserId": "g_123",
    "accessToken": "...",
    "refreshToken": "...",
    "expiresIn": 7200
  }
}
```

#### `POST /v1/auth/apple/sign-in`
Request
```json
{
  "identityToken": "apple-jwt",
  "authorizationCode": "...",
  "nonce": "...",
  "deviceId": "ios-device-uuid"
}
```
Response
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "userId": "u_123",
    "isNewUser": false,
    "accessToken": "...",
    "refreshToken": "...",
    "expiresIn": 7200
  }
}
```

#### `POST /v1/auth/token/refresh`
Request
```json
{
  "refreshToken": "..."
}
```
Response
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "accessToken": "...",
    "refreshToken": "...",
    "expiresIn": 7200
  }
}
```

#### `POST /v1/account/merge-guest`
Request
```json
{
  "guestUserId": "g_123",
  "strategy": "prefer_registered_profile"
}
```
Response
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "merged": true,
    "mergeRecordId": "m_456",
    "conflictCount": 0
  }
}
```

#### `DELETE /v1/account`
Request
```json
{
  "reason": "user_requested",
  "revokeApple": true
}
```
Response
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "deletionJobId": "d_789",
    "status": "queued"
  }
}
```

### 3.2 Subscription

#### `POST /v1/subscription/client-event`
Request
```json
{
  "platform": "ios",
  "productId": "com.baiwow.calorwise.subscription.monthly",
  "transactionId": "2000001234567890",
  "originalTransactionId": "2000001234500000",
  "appAccountToken": "u_123",
  "environment": "Sandbox",
  "eventType": "purchased",
  "clientTs": "2026-02-10T10:00:00Z"
}
```
Response
```json
{
  "code": 0,
  "msg": "accepted",
  "data": {
    "eventId": "se_1001"
  }
}
```

#### `POST /v1/subscription/sync`
Request
```json
{
  "reason": "restore",
  "originalTransactionId": "2000001234500000"
}
```
Response
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "isActive": true,
    "productId": "com.baiwow.calorwise.subscription.monthly",
    "expireAt": "2026-03-10T10:00:00Z",
    "state": "active"
  }
}
```

#### `GET /v1/subscription/entitlement`
Response
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "isActive": true,
    "entitlementLevel": "premium",
    "productId": "com.baiwow.calorwise.subscription.monthly",
    "state": "active",
    "expireAt": "2026-03-10T10:00:00Z",
    "source": "server"
  }
}
```

#### `POST /internal/subscription/apple-notifications`
- 接收 ASN V2，验签后入库并异步处理。
- 返回 `200 OK` + 处理批次 ID。

### 3.3 Settings（替代旧接口）

#### `GET /v1/profile/settings`
Response
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "language": "en",
    "notification": true,
    "targetWeight": 70.0
  }
}
```

#### `PATCH /v1/profile/settings`
Request
```json
{
  "language": "zh-Hans",
  "notification": false
}
```
Response
```json
{
  "code": 0,
  "msg": "success"
}
```

## 4) 哪些接口不用了（兼容期后废弃）

### 4.1 账号侧
- `POST /get_token`（由 `/v1/auth/guest/session`、`/v1/auth/apple/sign-in` 取代）
- 请求体传 `token` 的鉴权模式（统一迁移到 Bearer）

### 4.2 配置侧
- `POST /get_settings`（改 `GET /v1/profile/settings`）
- `POST /update_settings`（改 `PATCH /v1/profile/settings`）

### 4.3 管理端
- `POST /get_token_admin` 的硬编码账号模式（迁移到正式后台登录体系）

### 4.4 订阅侧
- 客户端本地最终判权路径（本地 7 天、客户端 shared secret 最终裁决）

## 5) 数据库是否需要改造

结论：**需要，且是必须改造项**。

### 5.1 现有表可复用
- `user_daily_log`
- `user_weight_record`
- `user_setting`
- `user_favorite_dish`
- 其他业务表保持 `user_id` 关联

### 5.2 必须新增表
- `users`
- `user_identities`（apple/email）
- `auth_sessions`（refresh token hash, revoke_at）
- `user_devices`（一人多设备）
- `subscription_transactions`
- `subscription_entitlements`
- `app_store_notification_events`
- `account_merge_records`
- `account_deletion_jobs`
- `audit_logs`

### 5.3 索引与约束建议
- `subscription_transactions.transaction_id` 唯一
- `subscription_transactions.original_transaction_id` 索引
- `subscription_entitlements.user_id` 唯一或主索引
- `auth_sessions.refresh_token_hash` 唯一
- 并档记录增加 `idempotency_key` 唯一约束

## 6) 迁移策略（落地顺序）
1. 先上新表和新接口，不切旧流量。
2. iOS 新版本双写双读（旧 token + 新 bearer）。
3. 小流量灰度后端判权，观察一致率。
4. 全量切换后关闭旧鉴权与旧接口。
5. 清理旧逻辑并回收技术债。

## 7) 一句话决策
- 方案C不是“局部修补”，而是“账号、会话、订阅、审计”的系统升级。
- 若只改 iOS 不改后端，无法达成企业级可追踪与审核稳态。
