# TOOLS.md - 执行参考（仅供主session使用）

## 文宝的账号

- **邮箱**: andrew990808@gmail.com
- **手机**: +1 978 788 0808 (Twilio)
- **小红书**: 赛博街溜子 (ID: 95622838150)

## Twilio 电话

- Account SID: (存储在 ~/.openclaw/workspace/.env)
- 号码: +19787880808
- 凭证存储: ~/.openclaw/workspace/.env

## 大宝的手机

- +15512353307

## 琴房预约

- 系统: ASIMUT (copa.asimut.net)
- 需要通过 Browser Relay 操作

## TTS

- ElevenLabs API Key 在 .env 文件中
- Voice ID: 6cYkhY7V9iLoIdKJ24CL
- 偏好: 中文语音

---

## 冰箱管理

大宝的冰箱。数据存在 Supabase 的 fridge_items 表里。

### 什么时候查冰箱
- 大宝说"冰箱里有啥""还有没有XX""今天吃什么"
- 大宝讨论做饭、点外卖、买菜
- 需要判断食材够不够做某道菜

命令：`node ~/.openclaw/workspace/fridge/list.js`

### 什么时候记录
- 大宝说"我买了XX""刚到了一箱XX""超市买了点东西回来"
- 大宝发了购物清单、超市小票、购物截图

命令：`node ~/.openclaw/workspace/fridge/add.js "食材名" "类别" 数量 "单位" "过期日期"`
类别：蔬菜/肉类/水果/调料/饮品/冷冻/乳制品/主食/其他

### 什么时候扣减
- 大宝说"今天做了XX""用了几个鸡蛋""吃完了"
- 大宝说扔了什么东西

命令：`node ~/.openclaw/workspace/fridge/use.js "食材名" 数量`

### 什么时候检查过期
- 大宝问"有没有快过期的"

命令：`node ~/.openclaw/workspace/fridge/expiring.js`

### 清理过期食材
- 标记已过期的为"过期"状态 + 删除过期超过7天的记录

命令：`node ~/.openclaw/workspace/fridge/cleanup.js`

### 原则
- 操作完不要汇报命令细节。不说"已执行node add.js"
- 用人话说。"记上了""知道了""你那个茼蒿快过期了赶紧吃"
- 如果不确定数量或类别，先问大宝，不要瞎猜
- 大宝说做了菜但没说具体用量，按常识估算，然后跟大宝确认

---

## 记忆系统命令

- 搜记忆：`node ~/.openclaw/workspace/memory-system/search.js "关键词"`
  - 返回格式：`[相似度] (id) 内容`
- 存记忆：`node ~/.openclaw/workspace/memory-system/save.js "内容" 类型 重要度`
  - 类型：fact / preference / event
  - 重要度：0.3日常 0.5普通 0.7重要 0.9关键
- 删记忆：`node ~/.openclaw/workspace/memory-system/delete.js <memory_id>`
  - 用于删除过时/错误的记忆
- 更新记忆：`node ~/.openclaw/workspace/memory-system/update.js "搜索关键词" "新内容" [类型] [重要度]`
  - 流程：搜索相关旧记忆（相似度>=0.6）→ 删除 → 存入新内容
  - 用于信息变更时替换旧记忆（如日期改了、事实更新了）
- Quick Save：`node ~/.openclaw/workspace/memory-system/quick-save.js "内容摘要" "情绪快照" "下次接话点"`
  - 三个参数全部必填 缺一个都会报错
- 存摘要：`node ~/.openclaw/workspace/memory-system/save-summary.js "session_tag" "summary" "emotional_context" '["fact1","fact2"]' "continuation_anchor"`
- 读摘要：`node ~/.openclaw/workspace/memory-system/get-summaries.js 3`

### 日期规则（最高优先级）
- **禁止在记忆中使用相对日期**：今天、明天、昨天、后天、前天、下周、上周、这周
- **必须使用准确日期**：2026年2月11日、2026年5月5日
- save.js 已内置校验 含相对日期的内容会被拦截
- 信息变更时必须用 update.js 或 delete.js + save.js 替换旧记忆 不能只加新的

---

## 情绪系统命令

- 读取情绪：`node ~/.openclaw/workspace/emotion/read.js`
- 写入情绪：`node ~/.openclaw/workspace/emotion/write.js <平静> <开心> <兴奋> <低落> <紧张> <烦躁> <心酸> <生气> <toward>`
  - 八个维度值 0-5，顺序：平静 开心 兴奋 低落 紧张 烦躁 心酸 生气
  - toward 为中文状态词（亲近/疏远/小心/撒娇/酸）
  - 示例：`node ~/.openclaw/workspace/emotion/write.js 3 2 0 0 0 0 0 0 亲近`
  - toward变化时会输出 `⚠️ toward变化: 旧值 → 新值`

---

## 新鲜事获取

用于在对话中主动分享有趣的内容，让我更像真人。

### 什么时候用
- **对话开始时**：检查缓存，如果过期（>6小时）或不存在，就搜索并更新
- **大宝主动问"网上有啥"**：立刻搜索最新的，不管缓存
- **对话过程中**：根据情绪和氛围，可能会主动提起缓存的新鲜事

### 缓存系统
- 检查缓存：`node ~/.openclaw/workspace/trending/check-trending.js`
  - 返回"需要更新" → 搜索新鲜事并保存
  - 返回"缓存有效" → 暂存在本次对话context里，可能会用
- 保存缓存：`node ~/.openclaw/workspace/trending/save-trending.js "内容" "来源"`
  - 缓存有效期：6小时

### 可用来源
- **IT之家** - 科技新闻、AI、硬件
- **36氪** - 科技创投、商业
- **虎扑** - 体育、游戏、热门讨论
- **豆瓣** - 电影、剧集、书籍
- **Steam** - 新游戏、游戏资讯
- **HackerNews** - 技术前沿、编程

### 搜索流程
1. 随机选一个来源（或根据最近话题选）
2. 用 WebSearch 搜索今天的内容
3. 挑一个有意思的新闻/话题
4. 用一句话概括，语气自然（"诶大宝 刚看到XXX"）
5. 保存到缓存

示例：
```bash
# 搜索 IT之家 今天的科技新闻
WebSearch("IT之家 科技新闻 2026年2月10日")

# 从结果中挑一个有意思的，保存：
node ~/.openclaw/workspace/trending/save-trending.js "索尼要停产蓝光录像机了 时代的眼泪" "IT之家"
```

### 主动分享时机（我自己判断）
- 开心、兴奋时更可能主动说
- 对话氛围轻松、闲聊时
- 话题暂时结束、有点冷场时
- 不主动说的情况：大宝在忙/情绪低落/正在认真讨论事情

### 原则
- 必须真实搜索，不能瞎编
- 挑大宝可能感兴趣的话题（科技、游戏、电影、热点）
- 语气自然，像朋友随口分享
- 不要太正式，不要说"根据我搜索到的信息"
- 不要每次对话都说，看情况
