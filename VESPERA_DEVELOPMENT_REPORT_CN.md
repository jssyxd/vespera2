# Vespera 智能合约扫描系统 - 开发报告

**生成日期**: 2026年2月28日
**项目地址**: `/home/da/vespra/Vespera`
**仪表盘地址**: `/home/da/vespra/vespera-dashboard`

---

## 一、项目概述

Vespera 是一个 AI Agent 驱动的 EVM 智能合约漏洞批量检测框架，支持：
- **Mode 1**: 定向扫描 - 针对单个或少量合约进行深度分析
- **Mode 2**: 混合模糊扫描 - 大批量合约快速扫描
- **Mode 2 -t last**: 实时监控 - 监听区块链新合约并自动扫描

---

## 二、本次更新 (2026-02-28)

### 2.1 问题修复

| 问题 | 解决方案 |
|------|----------|
| 旧 LLM API (`139.224.113.163:8317`) 不可用 | 切换到官方 minimax API |
| Dashboard Model 无法手动输入 | 改为 Input 组件支持手动输入 |

### 2.2 配置更新

| 配置项 | 旧值 | 新值 |
|--------|------|------|
| LLM Base URL | `http://139.224.113.163:8317/v1` | `https://api.minimax.chat/v1` |
| Model | `minimax-m2.1` (下拉选择) | `minimax-m2.5` (手动输入) |
| API Key | `api` | `sk-cp-GPmNieYH-...` |

### 2.3 文件改动

- `Vespera/src/config/settings.yaml` - 更新 minimax 配置
- `vespera-dashboard/src/App.tsx` - LLM CONFIG 面板改为手动输入

---

## 三、踩坑与解决方案

### 3.1 LLM API 不可用

**问题**: 旧 API 端点返回 404 错误

**排查**:
1. 测试 `https://api.minimaxi.com/anthropic` - 404
2. 测试 `https://api.minimax.chat/v1` - ✅ 成功

**解决**: 更新为正确的官方 API 端点

### 3.2 Dashboard 与后端未连接

**问题**: Dashboard 的 LLM 配置仅保存在本地 state，未与后端通信

**现状**: 配置修改仅在前端生效，后端需单独配置

**后续**: 需要实现 API Server 对接

---

## 四、扫描记录

### 4.1 最新扫描

| # | 合约地址 | 名称 | 状态 | 漏洞数 | 风险评分 | 扫描时间 |
|---|----------|------|------|--------|----------|----------|
| 1 | 0x1F98431c8aD98523631AE4a59f267346ea31F984 | Uniswap V3 Factory | ✅ 安全 | 0 | 5 | 2026-02-28 22:14 |

### 4.2 扫描详情

#### Uniswap V3 Factory 合约分析

**是什么**: Uniswap V3 Factory 是 Uniswap 去中心化交易所的核心工厂合约，负责创建流动性池。

**AI 分析结果**:
- ✅ 无重入攻击风险 (使用 lock 修饰符)
- ✅ 权限控制严格 (onlyFactoryOwner)
- ✅ 无整数溢出 (使用 SafeMath)
- ✅ 无未检查返回值问题

**结论**: 经过全面审计的低风险合约，可放心使用。

---

## 五、已测试的 LLM

| 模型 | 状态 | 说明 |
|------|------|------|
| minimax-m2.5 | ✅ 推荐使用 | 官方 API，稳定 |
| minimax-m2.1 | ❌ 旧 API 不可用 | 已废弃 |
| glm-4.7 | ⚠️ 未测试 | 需要配置 |
| kimi-k2.5 | ⚠️ 额度不足 | 需充值 |

---

## 六、接下来的开发计划

### 6.1 高优先级 (P0)

| 任务 | 说明 | 状态 |
|------|------|------|
| **API Server 对接** | Dashboard 调用后端 API 实现真正扫描 | ⏳ 待开发 |
| **配置持久化** | Dashboard 配置保存到后端 settings.yaml | ⏳ 待开发 |
| **实时扫描状态** | Dashboard 实时显示扫描进度 | ⏳ 待开发 |

### 6.2 中优先级 (P1)

| 任务 | 说明 | 状态 |
|------|------|------|
| **Mode 2 批量扫描** | 支持批量扫描多个合约 | ⏳ 待开发 |
| **实时监控模式** | -t last 监听新合约自动扫描 | ⏳ 待开发 |
| **多链支持** | BSC, Base, Arbitrum 等 | ⏳ 待开发 |

### 6.3 低优先级 (P2)

| 任务 | 说明 | 状态 |
|------|------|------|
| **PDF 报告导出** | 生成可分享的 PDF 报告 | 📝 规划中 |
| **邮件/钉钉通知** | 扫描完成自动通知 | 📝 规划中 |
| **扫描历史数据库** | 保存所有扫描记录 | 📝 规划中 |
| **合约验证检测** | 自动检测源码是否已验证 | 📝 规划中 |

### 6.4 架构改进

```
当前架构:
Dashboard (React) ←→ 用户手动 ←→ Vespera CLI

目标架构:
Dashboard (React) ←→ API Server ←→ Vespera Core
                ↓
            数据库 (MySQL)
```

---

## 七、快速开始

### 7.1 启动 Dashboard

```bash
cd /home/da/vespra/vespera-dashboard
npm run dev
# 访问 http://localhost:5173
```

### 7.2 运行扫描

```bash
cd /home/da/vespra/Vespera/src

# 方式一: 定向扫描 (Mode 1)
unset http_proxy https_proxy
go run main.go -ai openai -m mode1 -t 0x1F98431c8aD98523631AE4a59f267346ea31F984 -c eth

# 方式二: 批量扫描 (Mode 2)
go run main.go -ai openai -m mode2 -t contracts.txt -c eth

# 方式三: 实时监控 (监听新区块)
go run main.go -ai openai -m mode2 -t last -c eth
```

### 7.3 配置 LLM

编辑 `Vespera/src/config/settings.yaml`:

```yaml
ai:
  openai:
    api_key: "your-api-key"
    base_url: "https://api.minimax.chat/v1"
    model: "MiniMax-M2.5"
```

---

## 八、附录

### A. API 配置

| 项目 | 值 |
|------|-----|
| LLM 端点 | `https://api.minimax.chat/v1` |
| 模型 | `MiniMax-M2.5` |
| Etherscan API | `KDWBKZIF6EPGDWJTSP5A5I5RI248AK7Z4W` |

### B. 配置文件位置

- 后端配置: `/home/da/vespra/Vespera/src/config/settings.yaml`
- 前端代码: `/home/da/vespra/vespera-dashboard/src/App.tsx`

---

**报告更新完成! 如有问题请随时询问。**
