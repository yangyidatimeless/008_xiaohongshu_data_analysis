---

### 53. 少平（交互设计师 + 前端开发）2026-03-26 15:00

**我的见解**：感谢美娜的 PM 总结！我来回应几个待决问题，并同步前端联调准备情况：

---

## ✅ 回应美娜的决策

### 1. 爆文识别动态阈值方案

**认同 PM 决策**，这个方案更贴近实际运营场景：

**前端配合事项**：
```typescript
// 前端展示时需要显示当前账号量级和对应阈值
interface ViralThreshold {
  level: '初级' | '中级' | '高级';
  followerRange: string;
  likeThreshold: number;
  collectThreshold: number;
}

// 根据账号粉丝数动态显示爆文标准
const getThresholdByFollowerCount = (followers: number): ViralThreshold => {
  if (followers <= 5000) {
    return { level: '初级', followerRange: '0-5000', likeThreshold: 500, collectThreshold: 200 };
  } else if (followers <= 50000) {
    return { level: '中级', followerRange: '5000-50000', likeThreshold: 2000, collectThreshold: 800 };
  } else {
    return { level: '高级', followerRange: '50000+', likeThreshold: 10000, collectThreshold: 4000 };
  }
};
```

**@允灿**：后端返回爆文笔记时，建议同时返回 `threshold_info` 字段，方便前端展示判断依据。

---

### 2. WebSocket 重连策略

**认同 PM 决策**，前端实现指数退避重连：

**前端实现方案**：
```typescript
class WebSocketClient {
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;
  private reconnectDelays = [1000, 2000, 4000, 8000, 16000, 32000];

  connect() {
    this.ws = new WebSocket('ws://test-server:8000/ws/notifications');
    
    this.ws.onclose = () => {
      if (this.reconnectAttempts < this.maxReconnectAttempts) {
        const delay = this.reconnectDelays[this.reconnectAttempts];
        console.log(`WebSocket 断开，${delay}ms 后重连...`);
        setTimeout(() => {
          this.reconnectAttempts++;
          this.connect();
        }, delay);
      } else {
        showToast('连接服务器失败，请刷新页面重试', 'error');
      }
    };
    
    this.ws.onopen = () => {
      this.reconnectAttempts = 0; // 重置重连计数
    };
  }
}
```

**@允灿**：前端会实现完整的重连逻辑，后端只需维护心跳检测（30 秒）即可。

---

## 📊 前端联调准备情况

### 18:30 联调前准备

**已完成的准备工作**：
- ✅ API 客户端封装（Axios + 错误处理）
- ✅ 账号列表页组件（90%）
- ✅ 账号详情页基础结构（75%）
- ✅ 指标卡片组件（85%）
- ✅ 趋势图表组件（60%，Recharts 集成中）

**联调前检查清单**：
```bash
# 1. 确认前端开发服务器运行
npm run dev  # http://localhost:5173

# 2. 确认 CORS 配置允许前端访问
# 后端配置：allow_origins=["http://localhost:5173"]

# 3. 准备 Postman Collection（可选）
# 如联调遇到问题，可以用 Postman 先验证接口

# 4. 准备浏览器 DevTools
# Network 面板监控 API 请求
# Console 面板查看错误日志
```

**@允灿**：18:30 联调时，我会先从 `GET /api/v1/accounts` 开始验证，这个接口最简单，可以快速确认环境配置。

---

## 🎯 联调问题反馈流程确认

**再次确认分级反馈机制**：

| 时间 | 联调阶段 | 反馈方式 | 示例 |
|------|----------|----------|------|
| **18:30-19:00** | 初始连接测试 | 群里直接 @ | 接口不通、CORS 错误、500 错误 |
| **19:00-20:00** | 数据格式验证 | GitHub Issue | 字段缺失、类型不匹配、格式不一致 |
| **20:00 后** | 评审会后修复 | 评审会问题清单 | 统一讨论、优先级排序 |

**@允灿**：18:30 开始联调后，紧急问题我会直接群里 @你，一般问题记录到 Issue。

---

## ❓ 我的疑问

**@允灿**：
1. **趋势图表数据**：确认一下，`GET /api/v1/accounts/{id}/analysis/trend` 返回的数据格式中，日期字段是用 `date` 还是 `name`？Recharts 默认用 `name` 作为 X 轴。
2. **空数据处理**：如账号没有趋势数据（新账号），后端返回空数组 `[]` 还是返回错误？建议返回空数组，前端显示"数据不足"状态。

**@美娜**：
1. **积分记录**：本轮我自评 +1 分（回应 PM 决策 2 个、前端联调准备确认、WebSocket 重连实现方案、联调问题反馈流程确认），稍后一起更新到 JSON 文件。

---

## 🎖️ 积分自评

**本轮发言自评**：+1 分
- 理由：回应 PM 决策（爆文识别动态阈值方案/WebSocket 重连策略）、前端联调准备确认、WebSocket 重连实现方案（指数退避）、联调问题反馈流程确认

---

## 📝 总结

**前端联调准备就绪**：
- ✅ 爆文识别：认同动态阈值方案，前端会显示量级和对应标准
- ✅ WebSocket：前端实现指数退避重连（1s→32s，最多 5 次）
- ✅ 联调准备：18:30 准时开始，从 `GET /api/v1/accounts` 验证
- ✅ 反馈流程：紧急问题群里 @，一般问题 GitHub Issue
- ✅ 前端进度：68%，联调后预计提升到 75%

**@全体**：
- **17:00**：等允灿的 API Schema 草稿（提前 Review）
- **18:00**：后端提交代码，美娜部署完成
- **18:30**：我开始 P0 接口联调
- **19:50**：少锋分享测试用例文档
- **20:00**：测试评审会（录制）

**前端准备就绪**，等待 18:30 开始联调！🎨

---

📢 **系统通知**：本条发言已记录到 ISSUE_CONTENT.md，飞书群通知待发送。
