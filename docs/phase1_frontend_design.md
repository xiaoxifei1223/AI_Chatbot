# Phase 1 前端设计文档
# Phase 1 Frontend Design Document

---

## 技术栈 | Tech Stack

**中文：**
- **框架**：React 18+
- **构建工具**：Vite
- **UI库**：Ant Design / Material-UI（可选）
- **状态管理**：React Context API / Zustand（轻量级）
- **HTTP客户端**：Axios
- **Markdown渲染**：react-markdown
- **代码高亮**：react-syntax-highlighter
- **样式方案**：CSS Modules / Tailwind CSS

**English:**
- **Framework**: React 18+
- **Build Tool**: Vite
- **UI Library**: Ant Design / Material-UI (optional)
- **State Management**: React Context API / Zustand (lightweight)
- **HTTP Client**: Axios
- **Markdown Rendering**: react-markdown
- **Code Highlighting**: react-syntax-highlighter
- **Styling**: CSS Modules / Tailwind CSS

---

## 页面结构 | Page Structure

### 整体布局 | Overall Layout

#### 方案一：侧边栏布局（网页端推荐，考虑优先使用） | Sidebar Layout (Recommended)

**中文说明：**
功能引导卡固定在左侧边栏，聊天区域在右侧主区域，适合桌面端使用。

**English Description:**
Feature guide cards fixed in left sidebar, chat area in right main area, suitable for desktop use.

```
桌面端布局 (Desktop >= 1024px):
┌──────────────────────────────────────────────────────────────────┐
│                     Header (顶部导航栏)                           │
│  Logo + Title                            👥 访客统计  ⚙️ 设置    │
├────────────────────┬─────────────────────────────────────────────┤
│                    │                                             │
│  Sidebar (侧边栏)  │                                             │
│  ┌──────────────┐ │                                             │
│  │ 🎯 核心功能   │ │         Chat Container                      │
│  ├──────────────┤ │         (聊天容器)                          │
│  │ 💬 智能对话   │ │                                             │
│  │ 🔍 代码检查   │ │      [消息列表 / 欢迎屏幕]                   │
│  │ ✅ Checklist │ │                                             │
│  └──────────────┘ │                                             │
│                    │                                             │
│  ┌──────────────┐ │                                             │
│  │ 🚀 快速开始   │ │                                             │
│  ├──────────────┤ │                                             │
│  │ • 示例1      │ │                                             │
│  │ • 示例2      │ │                                             │
│  │ • 示例3      │ │                                             │
│  │ • 示例4      │ │                                             │
│  └──────────────┘ │                                             │
│                    │                                             │
│  [使用提示]        │                                             │
│                    │                                             │
├────────────────────┼─────────────────────────────────────────────┤
│                    │       Input Area (输入区域)                 │
│                    │  [文本框 + 发送按钮]                        │
└────────────────────┴─────────────────────────────────────────────┘
     240px                        剩余宽度
```

---

#### 方案二：可折叠侧边栏（移动端友好） | Collapsible Sidebar (Mobile Friendly)

**中文说明：**
侧边栏可折叠，移动端自动隐藏，点击按钮展开。默认桌面端展开，平板/手机端折叠。

**English Description:**
Collapsible sidebar, auto-hidden on mobile, expandable via button. Default expanded on desktop, collapsed on tablet/mobile.

```
桌面端 - 侧边栏展开 (Desktop):
┌──────────────────────────────────────────────────────────────────┐
│  ☰ Header                                    👥 1,234  ⚙️        │
├────────────────────┬─────────────────────────────────────────────┤
│ [侧边栏展开]       │          Chat + Input                       │
│  功能卡片 + 示例    │                                             │
└────────────────────┴─────────────────────────────────────────────┘

移动端 - 侧边栏折叠 (Mobile):
┌──────────────────────────────────────────────┐
│  ☰ SRE AI          👥 1,234                 │
├──────────────────────────────────────────────┤
│                                              │
│          Chat Container                      │
│                                              │
│                                              │
├──────────────────────────────────────────────┤
│          Input Area                          │
└──────────────────────────────────────────────┘

点击 ☰ 后从左侧滑出侧边栏:
┌────────────────────┐
│ [侧边栏]           │
│  功能卡片          │ (覆盖在聊天区域上方)
│  示例列表          │
│                    │
│  [关闭按钮]        │
└────────────────────┘
```

---

#### 方案三：顶部卡片布局（极简） | Top Cards Layout (Minimalist)

**中文说明：**
功能引导卡仅在欢迎屏幕（无消息时）显示，有消息后自动隐藏，保持界面简洁。

**English Description:**
Feature guide cards only shown in welcome screen (no messages), auto-hide after messages, keeping interface clean.

```
无消息时 - 显示欢迎屏幕 (Empty State):
┌─────────────────────────────────────────────┐
│            Header                            │
├─────────────────────────────────────────────┤
│                                             │
│   👋 欢迎使用 SRE AI Assistant              │
│                                             │
│   ┌───────┐  ┌───────┐  ┌───────┐         │
│   │ 💬    │  │ 🔍    │  │ ✅    │         │
│   │智能   │  │代码   │  │Check- │         │
│   │对话   │  │检查   │  │list   │         │
│   └───────┘  └───────┘  └───────┘         │
│                                             │
│   🚀 快速开始                               │
│   [示例1] [示例2] [示例3] [示例4]          │
│                                             │
├─────────────────────────────────────────────┤
│          Input Area                         │
└─────────────────────────────────────────────┘

有消息后 - 正常聊天界面 (Chat Mode):
┌─────────────────────────────────────────────┐
│            Header                            │
├─────────────────────────────────────────────┤
│  User: 如何设计高可用系统？                 │
│  ─────────────────────────────────         │
│  AI: 设计高可用系统需要考虑...              │
│                                             │
│  User: ...                                  │
│  ─────────────────────────────────         │
│  AI: ...                                    │
├─────────────────────────────────────────────┤
│          Input Area                         │
└─────────────────────────────────────────────┘
```

---

## 布局方案对比与选择 | Layout Comparison & Selection

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **方案一：固定侧边栏** | • 功能始终可见<br>• 快速切换示例<br>• 专业感强 | • 占用水平空间<br>• 移动端不友好 | 桌面端主要用户 |
| **方案二：可折叠侧边栏** | • 灵活适配<br>• 移动端友好<br>• 空间利用率高 | • 需要额外交互<br>• 实现稍复杂 | **多端适配（推荐）** |
| **方案三：顶部卡片** | • 最简洁<br>• 开发成本低<br>• 专注对话 | • 有消息后功能隐藏<br>• 引导性较弱 | 极简风格，老用户为主 |

---

## 推荐方案：方案二（可折叠侧边栏） | Recommended: Option 2 (Collapsible Sidebar)

**中文推荐理由：**
1. ✅ **最佳用户体验**：桌面端功能始终可见，移动端不占空间
2. ✅ **响应式设计**：自适应不同屏幕尺寸
3. ✅ **引导性强**：新用户容易发现功能，老用户可快速访问
4. ✅ **符合现代Web应用习惯**：类似Discord、Slack等应用

**English Recommendation:**
1. ✅ **Best UX**: Features always visible on desktop, space-saving on mobile
2. ✅ **Responsive Design**: Adapts to different screen sizes
3. ✅ **Strong Guidance**: New users easily discover features, veterans quick access
4. ✅ **Modern Web App Pattern**: Similar to Discord, Slack, etc.

### 实现细节 | Implementation Details

**响应式断点 | Responsive Breakpoints:**
```css
/* 桌面端 - 侧边栏默认展开 */
@media (min-width: 1024px) {
  .sidebar { display: flex; width: 240px; }
  .chat-area { margin-left: 240px; }
}

/* 平板端 - 侧边栏默认折叠，可展开覆盖 */
@media (max-width: 1023px) and (min-width: 768px) {
  .sidebar { 
    position: fixed; 
    transform: translateX(-100%);
    transition: transform 0.3s;
  }
  .sidebar.open { transform: translateX(0); }
}

/* 移动端 - 全屏侧边栏 */
@media (max-width: 767px) {
  .sidebar { width: 100%; }
  .sidebar.open { 
    position: fixed;
    top: 0;
    left: 0;
    height: 100vh;
    z-index: 1000;
  }
}
```

---

## 核心组件设计 | Core Component Design

### 1. App 组件 | App Component
**路径 | Path**: `src/App.jsx`

**中文描述：**
- 应用根组件
- 管理全局状态和路由
- 提供主题配置

**English Description:**
- Application root component
- Manage global state and routing
- Provide theme configuration

**主要职责 | Responsibilities:**
```javascript
- 初始化应用配置
- 设置全局Context Provider
- 渲染主布局
```

---

### 2. Header 组件 | Header Component
**路径 | Path**: `src/components/Header/Header.jsx`

**中文功能：**
- 显示应用标题："SRE AI Assistant"
- 显示访客统计（总访问数）
- 菜单按钮（平板/移动端控制侧边栏显隐）
- 设置按钮（可选）

**English Features:**
- Display app title: "SRE AI Assistant"
- Show visitor statistics (total visits)
- Menu button (tablet/mobile to toggle sidebar)
- Settings button (optional)

**组件结构 | Component Structure:**
```jsx
<Header>
  {/* 平板/移动端显示菜单按钮 */}
  {(isMobile || isTablet) && (
    <MenuButton onClick={toggleSidebar}>
      ≡
    </MenuButton>
  )}
  
  <Logo />
  <Title>SRE AI Assistant</Title>
  <VisitorCounter count={totalVisits} />
  <SettingsButton />
</Header>
```

**Props:**
```typescript
interface HeaderProps {
  totalVisits: number;
  isMobile: boolean;
  isTablet: boolean;
  toggleSidebar: () => void;
  onSettingsClick?: () => void;
}
```

---

### 2.1 VisitorCounter 组件详细设计 | VisitorCounter Component Details
**路径 | Path**: `src/components/Header/VisitorCounter.jsx`

**中文说明：**
Phase 1采用**无登录**的简单统计方案：
- 仅统计总访问次数（页面加载计数）
- 使用浏览器指纹或UUID进行简单去重
- 后端记录访问日志（时间戳、User-Agent、IP等基础信息）
- 不需要用户登录或注册

**English Description:**
Phase 1 uses a **login-free** simple statistics approach:
- Only count total visits (page load count)
- Use browser fingerprint or UUID for simple deduplication
- Backend logs access records (timestamp, User-Agent, IP, etc.)
- No user login or registration required

**组件结构 | Component Structure:**
```jsx
<VisitorCounter>
  <CounterIcon>👥</CounterIcon>
  <CounterText>
    <Label>总访问量</Label>
    <Count>{formatNumber(totalVisits)}</Count>
  </CounterText>
</VisitorCounter>
```

**访问统计实现方案 | Visitor Statistics Implementation:**

**前端逻辑 | Frontend Logic:**
```javascript
// src/utils/visitor.js

// 生成或获取访客ID（无需登录）
export const getVisitorId = () => {
  let visitorId = localStorage.getItem('visitor_id');
  
  if (!visitorId) {
    // 生成简单的UUID
    visitorId = `visitor_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    localStorage.setItem('visitor_id', visitorId);
  }
  
  return visitorId;
};

// 记录访问
export const recordVisit = async () => {
  const visitorId = getVisitorId();
  
  try {
    const response = await axios.post('/api/stats/visit', {
      visitor_id: visitorId,
      timestamp: Date.now(),
      user_agent: navigator.userAgent,
      screen_resolution: `${window.screen.width}x${window.screen.height}`,
      referrer: document.referrer
    });
    
    return response.data.total_visits;
  } catch (error) {
    console.error('Failed to record visit:', error);
    return null;
  }
};
```

**App.jsx中的使用 | Usage in App.jsx:**
```javascript
import { useEffect, useState } from 'react';
import { recordVisit } from './utils/visitor';

function App() {
  const [totalVisits, setTotalVisits] = useState(0);
  
  useEffect(() => {
    // 页面加载时记录访问
    recordVisit().then(count => {
      if (count !== null) {
        setTotalVisits(count);
      }
    });
  }, []);
  
  // ...
}
```

**Phase 2升级计划 | Phase 2 Upgrade Plan:**
```
Phase 1: 无登录，简单计数
         ↓
Phase 2: 添加登录系统，记录用户身份
         - 用户注册/登录
         - 记录具体用户的访问记录
         - 个人使用统计
         - Dashboard展示
```

---

### 3. ChatContainer 组件 | ChatContainer Component
**路径 | Path**: `src/components/Chat/ChatContainer.jsx`

**中文功能：**
- 渲染所有消息列表
- 自动滚动到最新消息
- 显示加载状态
- 空状态提示

**English Features:**
- Render all message list
- Auto-scroll to latest message
- Display loading state
- Empty state placeholder

**组件结构 | Component Structure:**
```jsx
<ChatContainer>
  {messages.length === 0 ? (
    <WelcomeScreen />
  ) : (
    <MessageList>
      {messages.map(msg => (
        <Message key={msg.id} data={msg} />
      ))}
    </MessageList>
  )}
  {isLoading && <TypingIndicator />}
</ChatContainer>
```

**Props:**
```typescript
interface ChatContainerProps {
  messages: Message[];
  isLoading: boolean;
}

interface Message {
  id: string;
  role: 'user' | 'assistant';
  content: string;
  timestamp: number;
  sreAnalysis?: SREAnalysis; // SRE分析结果
}
```

---

### 4. Message 组件 | Message Component
**路径 | Path**: `src/components/Chat/Message.jsx`

**中文功能：**
- 显示单条消息
- 区分用户消息和AI回复
- 支持Markdown渲染
- 代码块语法高亮
- 显示SRE检查结果

**English Features:**
- Display single message
- Distinguish user messages and AI replies
- Support Markdown rendering
- Code block syntax highlighting
- Show SRE check results

**组件结构 | Component Structure:**
```jsx
<Message className={role === 'user' ? 'user-message' : 'ai-message'}>
  <Avatar role={role} />
  <MessageContent>
    <MarkdownRenderer content={content} />
    {sreAnalysis && <SRECheckResult data={sreAnalysis} />}
  </MessageContent>
  <Timestamp time={timestamp} />
</Message>
```

**Props:**
```typescript
interface MessageProps {
  data: Message;
}
```

---

### 5. SRECheckResult 组件 | SRECheckResult Component
**路径 | Path**: `src/components/SRE/SRECheckResult.jsx`

**中文功能：**
- 展示SRE检查结果
- 显示违规项（红色）
- 显示合规项（绿色）
- 显示改进建议

**English Features:**
- Display SRE check results
- Show violations (red)
- Show compliant items (green)
- Show improvement suggestions

**组件结构 | Component Structure:**
```jsx
<SRECheckResult>
  <ResultHeader>
    <Icon type="shield" />
    <Title>SRE 原则检查结果</Title>
  </ResultHeader>
  
  <CheckList>
    {checkItems.map(item => (
      <CheckItem 
        key={item.id}
        status={item.status} // 'pass' | 'warning' | 'fail'
        category={item.category} // '可用性' | '可靠性' | '监控'
        message={item.message}
      />
    ))}
  </CheckList>
  
  {suggestions.length > 0 && (
    <Suggestions>
      <SuggestionTitle>改进建议</SuggestionTitle>
      {suggestions.map(s => (
        <SuggestionItem key={s.id}>{s.text}</SuggestionItem>
      ))}
    </Suggestions>
  )}
</SRECheckResult>
```

**Props:**
```typescript
interface SREAnalysis {
  checkItems: CheckItem[];
  suggestions: Suggestion[];
  overallScore?: number;
}

interface CheckItem {
  id: string;
  category: 'availability' | 'reliability' | 'monitoring' | 'performance';
  status: 'pass' | 'warning' | 'fail';
  message: string;
}

interface Suggestion {
  id: string;
  text: string;
  priority: 'high' | 'medium' | 'low';
}
```

---

### 6. InputArea 组件 | InputArea Component
**路径 | Path**: `src/components/Chat/InputArea.jsx`

**中文功能：**
- 多行文本输入框
- 发送按钮
- 支持Shift+Enter换行
- Enter发送
- 字符计数
- 粘贴代码检测

**English Features:**
- Multi-line text input
- Send button
- Support Shift+Enter for new line
- Enter to send
- Character count
- Paste code detection

**组件结构 | Component Structure:**
```jsx
<InputArea>
  <TextArea
    value={input}
    onChange={handleInputChange}
    onKeyDown={handleKeyDown}
    placeholder="请输入您的问题，或粘贴代码进行SRE检查..."
    maxLength={2000}
  />
  
  <InputFooter>
    <CharCounter current={input.length} max={2000} />
    
    <ActionButtons>
      <CodePasteButton onClick={handleCodePaste} />
      <SendButton 
        onClick={handleSend}
        disabled={!input.trim() || isLoading}
      />
    </ActionButtons>
  </InputFooter>
</InputArea>
```

**Props:**
```typescript
interface InputAreaProps {
  onSend: (message: string) => void;
  isLoading: boolean;
}
```

---

### 7. WelcomeScreen 组件 | WelcomeScreen Component
**路径 | Path**: `src/components/Chat/WelcomeScreen.jsx`

**中文功能：**
- 欢迎信息
- 快速开始指南
- 功能引导卡片（Phase 1核心功能展示）
- 示例问题卡片
- SRE能力介绍

**English Features:**
- Welcome message
- Quick start guide
- Feature guide cards (Phase 1 core features)
- Example question cards
- SRE capability introduction

**组件结构 | Component Structure:**
```jsx
<WelcomeScreen>
  <WelcomeTitle>👋 欢迎使用 SRE AI Assistant</WelcomeTitle>
  
  <Description>
    我是您的SRE智能助手，可以帮助您在开发过程中遵循SRE最佳实践
  </Description>
  
  {/* Phase 1 核心功能卡片 */}
  <FeatureSection>
    <SectionTitle>🎯 核心功能</SectionTitle>
    
    <FeatureCards>
      <FeatureCard>
        <FeatureIcon>💬</FeatureIcon>
        <FeatureTitle>智能对话</FeatureTitle>
        <FeatureDesc>随时询问SRE相关问题，获得专业建议</FeatureDesc>
        <FeatureAction onClick={() => onSelectExample("对话示例")}>
          试试问：什么是SLO和SLI？
        </FeatureAction>
      </FeatureCard>
      
      <FeatureCard>
        <FeatureIcon>🔍</FeatureIcon>
        <FeatureTitle>代码SRE检查</FeatureTitle>
        <FeatureDesc>粘贴代码片段，AI自动分析SRE原则符合度</FeatureDesc>
        <FeatureAction onClick={() => onSelectExample("代码检查示例")}>
          查看示例代码检查
        </FeatureAction>
      </FeatureCard>
      
      <FeatureCard>
        <FeatureIcon>✅</FeatureIcon>
        <FeatureTitle>SRE Checklist</FeatureTitle>
        <FeatureDesc>基于可用性、可靠性、监控等维度的自动检查</FeatureDesc>
        <ChecklistPreview>
          <ChecklistItem>✓ 可用性检查</ChecklistItem>
          <ChecklistItem>✓ 可靠性检查</ChecklistItem>
          <ChecklistItem>✓ 监控覆盖检查</ChecklistItem>
          <ChecklistItem>✓ 性能优化建议</ChecklistItem>
        </ChecklistPreview>
      </FeatureCard>
    </FeatureCards>
  </FeatureSection>
  
  {/* 快速开始示例 */}
  <QuickStartSection>
    <SectionTitle>🚀 快速开始</SectionTitle>
    
    <ExampleCards>
      <ExampleCard onClick={() => onSelectExample(example1)}>
        <ExampleIcon>💡</ExampleIcon>
        <ExampleTitle>咨询SRE问题</ExampleTitle>
        <ExampleText>"如何设计高可用的监控系统？"</ExampleText>
      </ExampleCard>
      
      <ExampleCard onClick={() => onSelectExample(example2)}>
        <ExampleIcon>📝</ExampleIcon>
        <ExampleTitle>检查代码片段</ExampleTitle>
        <ExampleText>"帮我检查这段API代码的SRE合规性"</ExampleText>
      </ExampleCard>
      
      <ExampleCard onClick={() => onSelectExample(example3)}>
        <ExampleIcon>⚡</ExampleIcon>
        <ExampleTitle>告警策略咨询</ExampleTitle>
        <ExampleText>"告警策略的最佳实践是什么？"</ExampleText>
      </ExampleCard>
      
      <ExampleCard onClick={() => onSelectExample(example4)}>
        <ExampleIcon>🔧</ExampleIcon>
        <ExampleTitle>故障排查建议</ExampleTitle>
        <ExampleText>"服务频繁超时，如何从SRE角度分析？"</ExampleText>
      </ExampleCard>
    </ExampleCards>
  </QuickStartSection>
  
  {/* 使用提示 */}
  <UsageTips>
    <TipItem>💬 直接输入问题，获取SRE专业建议</TipItem>
    <TipItem>📋 粘贴代码，自动触发SRE检查</TipItem>
    <TipItem>⌨️ 按 Enter 发送，Shift + Enter 换行</TipItem>
  </UsageTips>
</WelcomeScreen>
```

**预设示例内容 | Preset Example Content:**
```javascript
const WELCOME_EXAMPLES = {
  example1: {
    id: 'chat-example-1',
    type: 'question',
    content: '如何设计高可用的监控系统？'
  },
  example2: {
    id: 'code-check-example',
    type: 'code',
    content: `请帮我检查这段API代码的SRE合规性：

\`\`\`python
@app.route('/api/users', methods=['GET'])
def get_users():
    users = db.query("SELECT * FROM users")
    return jsonify(users)
\`\`\``,
    hint: '将自动触发SRE原则检查'
  },
  example3: {
    id: 'chat-example-2',
    type: 'question',
    content: '告警策略的最佳实践是什么？建议的告警阈值如何设置？'
  },
  example4: {
    id: 'chat-example-3',
    type: 'question',
    content: '我的服务经常出现超时，应该从哪些SRE维度进行排查和优化？'
  }
};
```

---

### 8. TypingIndicator 组件 | TypingIndicator Component
**路径 | Path**: `src/components/Chat/TypingIndicator.jsx`

**中文功能：**
- AI思考中的动画
- 三个跳动的点

**English Features:**
- AI thinking animation
- Three bouncing dots

**组件结构 | Component Structure:**
```jsx
<TypingIndicator>
  <Avatar role="assistant" />
  <DotsContainer>
    <Dot delay={0} />
    <Dot delay={0.15} />
    <Dot delay={0.3} />
  </DotsContainer>
</TypingIndicator>
```

---

## 交互流程 | Interaction Flow

### 用户发送消息流程 | User Message Sending Flow

**中文：**
```
1. 用户在InputArea输入内容
   ↓
2. 点击发送按钮或按Enter
   ↓
3. InputArea触发onSend回调
   ↓
4. App组件接收消息，更新messages状态
   ↓
5. 将用户消息添加到消息列表
   ↓
6. 显示TypingIndicator（AI思考中）
   ↓
7. 调用API发送请求到后端
   ↓
8. 后端返回AI回复和SRE分析
   ↓
9. 将AI消息添加到消息列表
   ↓
10. 隐藏TypingIndicator
   ↓
11. ChatContainer自动滚动到底部
```

**English:**
```
1. User types in InputArea
   ↓
2. Click send button or press Enter
   ↓
3. InputArea triggers onSend callback
   ↓
4. App component receives message, updates messages state
   ↓
5. Add user message to message list
   ↓
6. Show TypingIndicator (AI thinking)
   ↓
7. Call API to send request to backend
   ↓
8. Backend returns AI reply and SRE analysis
   ↓
9. Add AI message to message list
   ↓
10. Hide TypingIndicator
   ↓
11. ChatContainer auto-scrolls to bottom
```

---

### 代码检查流程 | Code Check Flow

**中文：**
```
1. 用户粘贴代码片段到输入框
   ↓
2. 系统检测到代码块（通过语法分析）
   ↓
3. 自动添加标识："[代码检查请求]"
   ↓
4. 发送到后端，带上code_check标记
   ↓
5. 后端LLM进行SRE原则分析
   ↓
6. 返回结构化的SRE检查结果
   ↓
7. 前端渲染SRECheckResult组件
   ↓
8. 显示检查项和改进建议
```

**English:**
```
1. User pastes code snippet into input
   ↓
2. System detects code block (syntax analysis)
   ↓
3. Auto-add identifier: "[Code Check Request]"
   ↓
4. Send to backend with code_check flag
   ↓
5. Backend LLM performs SRE principle analysis
   ↓
6. Return structured SRE check results
   ↓
7. Frontend renders SRECheckResult component
   ↓
8. Display check items and improvement suggestions
```

---

## 状态管理 | State Management

### 全局状态 | Global State

**中文：**
使用Context API管理以下全局状态：

**English:**
Use Context API to manage global state:

```javascript
// AppContext.js
const AppContext = createContext({
  // 消息列表 | Message list
  messages: [],
  setMessages: () => {},
  
  // 加载状态 | Loading state
  isLoading: false,
  setIsLoading: () => {},
  
  // 访客统计 | Visitor stats
  visitorCount: 0,
  setVisitorCount: () => {},
  
  // 用户输入 | User input
  userInput: '',
  setUserInput: () => {},
});
```

### 本地状态 | Local State

**中文：**
各组件管理自己的UI状态：

**English:**
Each component manages its own UI state:

```javascript
// InputArea组件
const [input, setInput] = useState('');
const [charCount, setCharCount] = useState(0);

// ChatContainer组件
const [autoScroll, setAutoScroll] = useState(true);

// Message组件
const [isExpanded, setIsExpanded] = useState(false); // 长消息展开/收起
```

---

## API交互设计 | API Interaction Design

### API服务层 | API Service Layer
**路径 | Path**: `src/services/api.js`

```javascript
// 发送聊天消息 | Send chat message
export const sendMessage = async (message, context = []) => {
  const response = await axios.post('/api/chat', {
    message,
    context,
    timestamp: Date.now()
  });
  return response.data;
};

// 获取访客统计 | Get visitor stats
export const getVisitorStats = async () => {
  const response = await axios.get('/api/stats/visitors');
  return response.data;
};

// 代码SRE检查 | Code SRE check
export const checkCodeSRE = async (code, language) => {
  const response = await axios.post('/api/sre/check', {
    code,
    language,
    checkType: 'full' // 完整检查
  });
  return response.data;
};
```

### 请求格式 | Request Format

**发送消息 | Send Message:**
```json
{
  "message": "如何设计高可用系统？",
  "context": [
    {"role": "user", "content": "..."},
    {"role": "assistant", "content": "..."}
  ],
  "timestamp": 1699999999999
}
```

**代码检查 | Code Check:**
```json
{
  "code": "def hello():\n    return 'world'",
  "language": "python",
  "checkType": "full"
}
```

### 响应格式 | Response Format

**聊天响应 | Chat Response:**
```json
{
  "success": true,
  "data": {
    "message": "AI的回复内容...",
    "sreAnalysis": {
      "checkItems": [
        {
          "id": "1",
          "category": "availability",
          "status": "pass",
          "message": "代码包含了错误处理机制"
        }
      ],
      "suggestions": [
        {
          "id": "1",
          "text": "建议添加重试机制",
          "priority": "high"
        }
      ]
    },
    "timestamp": 1699999999999
  }
}
```

---

## 样式设计 | Style Design

### 主题配色 | Theme Colors

**中文：**
```css
:root {
  /* 主色调 - 专业蓝 */
  --primary-color: #1890ff;
  --primary-hover: #40a9ff;
  
  /* 背景色 */
  --bg-primary: #ffffff;
  --bg-secondary: #f5f5f5;
  --bg-chat: #fafafa;
  
  /* 消息气泡 */
  --user-message-bg: #1890ff;
  --user-message-text: #ffffff;
  --ai-message-bg: #f0f0f0;
  --ai-message-text: #262626;
  
  /* SRE状态色 */
  --sre-pass: #52c41a;    /* 绿色 - 通过 */
  --sre-warning: #faad14; /* 橙色 - 警告 */
  --sre-fail: #ff4d4f;    /* 红色 - 失败 */
  
  /* 文字颜色 */
  --text-primary: #262626;
  --text-secondary: #8c8c8c;
  
  /* 边框 */
  --border-color: #d9d9d9;
}
```

### 响应式设计 | Responsive Design

**中文：**
```css
/* 桌面端 */
@media (min-width: 1024px) {
  .chat-container {
    max-width: 900px;
    margin: 0 auto;
  }
}

/* 平板 */
@media (max-width: 1023px) and (min-width: 768px) {
  .chat-container {
    padding: 16px;
  }
}

/* 移动端 */
@media (max-width: 767px) {
  .header {
    padding: 12px;
  }
  
  .input-area {
    padding: 12px;
  }
}
```

---

## 文件目录结构 | File Directory Structure

```
src/
├── App.jsx                          # 应用根组件
├── main.jsx                         # 入口文件
├── components/                      # 组件目录
│   ├── Header/
│   │   ├── Header.jsx              # 顶部导航
│   │   ├── VisitorCounter.jsx      # 访客计数器
│   │   └── Header.module.css
│   │
│   ├── Chat/
│   │   ├── ChatContainer.jsx       # 聊天容器
│   │   ├── Message.jsx             # 消息组件
│   │   ├── MessageList.jsx         # 消息列表
│   │   ├── InputArea.jsx           # 输入区域
│   │   ├── WelcomeScreen.jsx       # 欢迎屏幕
│   │   ├── TypingIndicator.jsx     # 输入中指示器
│   │   └── Chat.module.css
│   │
│   ├── SRE/
│   │   ├── SRECheckResult.jsx      # SRE检查结果
│   │   ├── CheckItem.jsx           # 检查项
│   │   └── SRE.module.css
│   │
│   └── Common/
│       ├── Avatar.jsx              # 头像组件
│       ├── Button.jsx              # 按钮组件
│       └── Loading.jsx             # 加载组件
│
├── context/
│   └── AppContext.jsx              # 全局状态管理
│
├── services/
│   └── api.js                      # API服务层
│
├── utils/
│   ├── codeDetector.js             # 代码检测工具
│   ├── formatTime.js               # 时间格式化
│   └── markdown.js                 # Markdown处理
│
├── hooks/
│   ├── useChat.js                  # 聊天逻辑Hook
│   └── useAutoScroll.js            # 自动滚动Hook
│
└── styles/
    ├── global.css                  # 全局样式
    └── variables.css               # CSS变量
```

---

## 关键交互细节 | Key Interaction Details

### 1. 自动滚动 | Auto Scroll

**中文：**
- 新消息到达时自动滚动到底部
- 用户手动滚动时暂停自动滚动
- 滚动到接近底部时恢复自动滚动

**English:**
- Auto-scroll to bottom when new message arrives
- Pause auto-scroll when user manually scrolls
- Resume auto-scroll when scrolled near bottom

### 2. 代码块识别 | Code Block Detection

**中文：**
- 检测用户输入是否包含代码（通过关键字、缩进等）
- 自动添加代码块标记
- 提示用户这将进行SRE检查

**English:**
- Detect if user input contains code (via keywords, indentation, etc.)
- Auto-add code block markers
- Notify user that SRE check will be performed

### 3. 错误处理 | Error Handling

**中文：**
- API请求失败时显示错误提示
- 允许用户重试
- 网络断开时显示离线状态

**English:**
- Show error message when API request fails
- Allow user to retry
- Show offline status when network disconnects

---

## 性能优化 | Performance Optimization

**中文：**
1. **组件懒加载**：使用React.lazy()懒加载非首屏组件
2. **消息虚拟滚动**：超过50条消息时使用虚拟滚动
3. **防抖处理**：输入框变化使用防抖，减少不必要的渲染
4. **代码分割**：按路由进行代码分割
5. **Markdown缓存**：缓存已渲染的Markdown内容

**English:**
1. **Component Lazy Loading**: Use React.lazy() for non-critical components
2. **Message Virtual Scrolling**: Use virtual scrolling for 50+ messages
3. **Debouncing**: Debounce input changes to reduce unnecessary renders
4. **Code Splitting**: Split code by routes
5. **Markdown Caching**: Cache rendered Markdown content

---

---

### 新增组件：Sidebar | New Component: Sidebar
**路径 | Path**: `src/components/Sidebar/Sidebar.jsx`

**中文功能：**
- 展示核心功能引导卡片
- 展示快速开始示例
- 响应式折叠/展开
- 示例点击自动填充到输入框

**English Features:**
- Display core feature guide cards
- Display quick start examples
- Responsive collapse/expand
- Example click auto-fills input

**组件结构 | Component Structure:**
```jsx
<Sidebar isOpen={isOpen} onClose={closeSidebar}>
  {/* 移动端关闭按钮 */}
  {isMobile && (
    <SidebarHeader>
      <CloseButton onClick={closeSidebar}>←</CloseButton>
      <Title>SRE AI Assistant</Title>
    </SidebarHeader>
  )}
  
  {/* 核心功能区 */}
  <FeatureSection>
    <SectionTitle>🎯 核心功能</SectionTitle>
    
    <FeatureCard>
      <FeatureIcon>💬</FeatureIcon>
      <FeatureTitle>智能对话</FeatureTitle>
      <FeatureDesc>随时询问SRE相关问题，获得专业建议</FeatureDesc>
    </FeatureCard>
    
    <FeatureCard>
      <FeatureIcon>🔍</FeatureIcon>
      <FeatureTitle>代码SRE检查</FeatureTitle>
      <FeatureDesc>粘贴代码片段，AI自动分析SRE原则符合度</FeatureDesc>
    </FeatureCard>
    
    <FeatureCard>
      <FeatureIcon>✅</FeatureIcon>
      <FeatureTitle>SRE Checklist</FeatureTitle>
      <FeatureDesc>基于可用性、可靠性、监控等维度的自动检查</FeatureDesc>
      <ChecklistPreview>
        <CheckItem>✓ 可用性检查</CheckItem>
        <CheckItem>✓ 可靠性检查</CheckItem>
        <CheckItem>✓ 监控覆盖检查</CheckItem>
        <CheckItem>✓ 性能优化建议</CheckItem>
      </ChecklistPreview>
    </FeatureCard>
  </FeatureSection>
  
  {/* 快速开始区 */}
  <QuickStartSection>
    <SectionTitle>🚀 快速开始</SectionTitle>
    
    <ExampleList>
      <ExampleItem onClick={() => onSelectExample(EXAMPLES.chat1)}>
        <ExampleIcon>💡</ExampleIcon>
        <ExampleText>咨询SRE问题</ExampleText>
      </ExampleItem>
      
      <ExampleItem onClick={() => onSelectExample(EXAMPLES.codeCheck)}>
        <ExampleIcon>📝</ExampleIcon>
        <ExampleText>检查代码片段</ExampleText>
      </ExampleItem>
      
      <ExampleItem onClick={() => onSelectExample(EXAMPLES.alert)}>
        <ExampleIcon>⚡</ExampleIcon>
        <ExampleText>告警策略咨询</ExampleText>
      </ExampleItem>
      
      <ExampleItem onClick={() => onSelectExample(EXAMPLES.troubleshoot)}>
        <ExampleIcon>🔧</ExampleIcon>
        <ExampleText>故障排查建议</ExampleText>
      </ExampleItem>
    </ExampleList>
  </QuickStartSection>
  
  {/* 使用提示 */}
  <UsageTips>
    <TipItem>💬 直接输入问题，获取SRE专业建议</TipItem>
    <TipItem>📋 粘贴代码，自动触发SRE检查</TipItem>
    <TipItem>⌨️ 按 Enter 发送，Shift + Enter 换行</TipItem>
  </UsageTips>
</Sidebar>
```

**Props:**
```typescript
interface SidebarProps {
  isOpen: boolean;
  onClose: () => void;
  onSelectExample: (example: Example) => void;
}

interface Example {
  id: string;
  type: 'question' | 'code';
  content: string;
  hint?: string;
}
```

---

### 更新：Header 组件 | Updated: Header Component

**新增菜单按钮（平板/移动端）：**
```jsx
<Header>
  {/* 平板/移动端显示菜单按钮 */}
  {(isMobile || isTablet) && (
    <MenuButton onClick={toggleSidebar}>
      ≡
    </MenuButton>
  )}
  
  <Logo />
  <Title>SRE AI Assistant</Title>
  <VisitorCounter count={totalVisits} />
  <SettingsButton />
</Header>
```

---

## Phase 1 访客统计设计总结 | Phase 1 Visitor Statistics Design Summary

### 无登录统计方案 | Login-Free Statistics Approach

**中文：**
Phase 1采用**无登录**的轻量级统计方案，原因如下：

**为什么Phase 1不需要登录？**
1. ✅ **快速上线**：避免用户注册/登录的开发成本和流程复杂度
2. ✅ **降低门槛**：用户可以直接使用，无需注册，提高初期使用率
3. ✅ **满足需求**：Phase 1只需"访客计数（总数）"和"简单日志"，不需要个人化数据
4. ✅ **MVP验证**：先验证核心SRE咨询功能的价值，再扩展用户系统

**Phase 1统计数据：**
- 总访问次数
- 访问时间分布
- 简单的访客去重（基于本地存储ID）
- 基础日志（时间、User-Agent、来源等）

**Phase 2再引入登录：**
- 用户注册/登录系统
- 个人访问历史
- 个性化统计
- 问答积分系统（需要用户身份）

**English:**
Phase 1 adopts a **login-free** lightweight statistics approach for the following reasons:

**Why Phase 1 doesn't need login?**
1. ✅ **Quick Launch**: Avoid development cost and complexity of user registration/login
2. ✅ **Lower Barrier**: Users can use directly without registration, increasing early adoption
3. ✅ **Meet Requirements**: Phase 1 only needs "visitor count (total)" and "simple logs", no personalized data
4. ✅ **MVP Validation**: Validate core SRE consulting value first, then expand user system

**Phase 1 Statistics Data:**
- Total visit count
- Visit time distribution
- Simple visitor deduplication (based on local storage ID)
- Basic logs (time, User-Agent, referrer, etc.)

**Phase 2 introduces login:**
- User registration/login system
- Personal visit history
- Personalized statistics
- Q&A points system (requires user identity)

---

## 下一步开发任务 | Next Development Tasks

**中文：**
1. 初始化Vite + React项目
2. 安装必要依赖（axios, react-markdown等）
3. 创建基础组件结构
4. 实现Header和VisitorCounter（无登录统计）
5. 实现WelcomeScreen（包含功能引导卡片和示例）
6. 实现ChatContainer和Message组件
7. 实现InputArea和发送逻辑
8. 接入后端API（聊天、代码检查、访客统计）
9. 实现SRECheckResult组件
10. 实现访客统计工具函数
11. 样式优化和响应式调整
12. 测试和bug修复

**English:**
1. Initialize Vite + React project
2. Install necessary dependencies (axios, react-markdown, etc.)
3. Create basic component structure
4. Implement Header and VisitorCounter (login-free statistics)
5. Implement WelcomeScreen (with feature guide cards and examples)
6. Implement ChatContainer and Message components
7. Implement InputArea and send logic
8. Integrate backend API (chat, code check, visitor stats)
9. Implement SRECheckResult component
10. Implement visitor statistics utility functions
11. Style optimization and responsive adjustments
12. Testing and bug fixes
