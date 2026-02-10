# CalorWise 账号体系与订阅后端依赖

Date: 2026-02-10

## Project context
- Project: CalorWise iOS app
- Context: 讨论“如何避免订阅与审核拒审”时，对现有账号体系进行盘点并提出落地迁移路径。

## Current-state signals
- 当前主身份接近“设备 token + 本地登录态”，而非完整服务端账号会话。
- 登录流程仍有 mock 痕迹（本地固定邮箱/密码校验），不属于真实鉴权。
- 订阅判权存在本地逻辑（如本地 7 天免费、客户端持有 shared secret），存在合规与安全风险。
- 网络层仍有 HTTP/ATS 放开配置，不适合正式账号与支付链路。

## Key learning
- 最稳演进方式不是“一步重写”，而是采用“双轨身份”：
  - 游客轨：保留 device token（降低首开门槛）。
  - 正式账号轨：Apple 登录/邮箱登录后签发服务端会话。
- 通过“游客数据并档”实现平滑迁移：登录后把饮食记录、体重、设置、收藏从 guest 归并到 user。

## Subscription-specific takeaway
- 订阅支付必须继续由 StoreKit 系统弹窗触发。
- 免费试用应在 App Store Connect 通过 Intro Offer 配置，不能在客户端本地放行。
- 订阅权益最终应以后端为准，客户端负责购买触发与展示。

## Backend-required capabilities
- 账号与会话：Apple token 校验、邮箱鉴权、session 签发与刷新。
- 账户合并：guest->user 数据并档接口与幂等策略。
- 订阅中台：交易入库、订阅状态聚合、到期/宽限/退款状态计算。
- 苹果服务端通知：接入 App Store Server Notifications v2。
- 合规能力：App 内删除账号对应的后端删除流程与审计。

## Practical decision rule
- 可先保留客户端可独立实现项（支付拉起、恢复购买入口、文案与引导）。
- 只要涉及“最终权限判定、跨设备一致性、退款与续费状态”，必须后端兜底。
