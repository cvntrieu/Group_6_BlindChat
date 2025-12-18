# BlindChat - Fullstack Web Application Summary

## Tổng quan

**BlindChat** là một ứng dụng web hỗ trợ người khiếm thị thông qua giao diện giọng nói AI. Ứng dụng cho phép người dùng tương tác với trợ lý ảo bằng giọng nói để thực hiện các tác vụ như đọc file, mô tả hình ảnh từ camera, và điều khiển giao diện.

---

## Kiến trúc hệ thống

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              Frontend                                    │
│                    (Next.js 15 + React 19 + LiveKit)                    │
│         Giao diện giọng nói, video streaming, chat transcript           │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
                    ▼                       ▼
┌───────────────────────────┐   ┌───────────────────────────────┐
│       AI Agent            │   │         Backend API           │
│   (Python + LiveKit)      │   │    (ASP.NET Core 9 + SQL)     │
│                           │   │                               │
│ • Voice AI Assistant      │◄──│ • User Authentication         │
│ • OpenAI GPT-4o-mini      │   │ • Conversation History        │
│ • Azure STT/TTS           │   │ • Message Storage             │
│ • Vision Processing       │   │                               │
└───────────────────────────┘   └───────────────────────────────┘
```

---

## 1. AI Agent (Python)

### Công nghệ sử dụng
- **LiveKit Agents SDK**: Real-time voice communication
- **OpenAI GPT-4o-mini**: Language model cho xử lý ngôn ngữ tự nhiên
- **Azure Speech Services**: Speech-to-Text (STT) và Text-to-Speech (TTS)
- **Silero VAD**: Voice Activity Detection

### Chức năng chính

#### 1.1 Voice Assistant (`agent.py`)
Trợ lý giọng nói AI với các công cụ:

| Tool | Mô tả |
|------|-------|
| `get_current_date_and_time` | Trả về ngày giờ hiện tại |
| `process_file_request` | Đọc và tóm tắt file PDF từ thư mục Downloads |
| `describe_camera_view` | Mô tả hình ảnh từ camera của người dùng |
| `control_ui_device` | Điều khiển UI (camera, microphone, chat panel) |

#### 1.2 File Processing (`functions.py`)
- **Route Request**: Phân loại yêu cầu người dùng (đọc raw text / tóm tắt)
- **Handle Summarization**: Tóm tắt nội dung văn bản
- **Handle Image Description**: Mô tả hình ảnh sử dụng OpenAI Vision API
- **File Reading**: Hỗ trợ các định dạng: `.pdf`, `.docx`, `.txt`, `.md`, `.csv`, `.json`

#### 1.3 Conversation Cache (`storage.py`)
- Lưu trữ tạm thời tin nhắn trong bộ nhớ
- Tự động đồng bộ với Backend sau mỗi 5 cặp tin nhắn (user + bot)
- Lấy lịch sử hội thoại để cung cấp context cho AI

#### 1.4 Utilities (`utils.py`)
- Tìm kiếm file PDF gần đây trong thư mục Downloads
- Đọc nội dung các loại file khác nhau
- Tự động đăng ký/đăng nhập user với Backend

---

## 2. Backend API (ASP.NET Core 9)

### Công nghệ sử dụng
- **ASP.NET Core 9**: Web API framework
- **Entity Framework Core**: ORM
- **SQL Server**: Database
- **ASP.NET Identity**: User management
- **JWT**: Authentication tokens

### Database Models

```
┌──────────────────┐    ┌─────────────────────────┐    ┌──────────────────┐
│      User        │    │  ConversationHistory    │    │     Message      │
├──────────────────┤    ├─────────────────────────┤    ├──────────────────┤
│ Id (IdentityUser)│◄───│ UserId                  │◄───│ ConversationId   │
│ UserName         │    │ CreatedAt               │    │ SenderType (0/1) │
│ ConversationHist │    │ Messages[]              │    │ Content          │
└──────────────────┘    └─────────────────────────┘    │ CreatedAt        │
                                                       └──────────────────┘
                                                       
SenderType: 0 = User, 1 = Bot
```

### API Endpoints

#### Account Controller (`/api/account`)
| Method | Endpoint | Mô tả |
|--------|----------|-------|
| POST | `/register` | Đăng ký user mới |
| POST | `/login` | Đăng nhập, trả về JWT token |

#### Message Controller (`/api/messages`)
| Method | Endpoint | Mô tả |
|--------|----------|-------|
| POST | `/` | Thêm danh sách tin nhắn mới |

#### Conversation History Controller (`/api/conversation-history`)
| Method | Endpoint | Mô tả |
|--------|----------|-------|
| GET | `/?limit=N` | Lấy N tin nhắn gần nhất của user |

---

## 3. Frontend (Next.js 15)

### Công nghệ sử dụng
- **Next.js 15**: React framework với App Router
- **React 19**: UI library
- **LiveKit Client SDK**: Real-time voice/video communication
- **TailwindCSS 4**: Styling
- **Motion (Framer Motion)**: Animations
- **Radix UI**: Accessible UI components

### Cấu trúc Components

```
components/
├── app/
│   ├── app.tsx              # Main app wrapper
│   ├── session-view.tsx     # Giao diện phiên voice chat
│   ├── welcome-view.tsx     # Màn hình chào mừng
│   ├── chat-transcript.tsx  # Hiển thị lịch sử chat
│   ├── tile-layout.tsx      # Layout video tiles
│   └── view-controller.tsx  # Điều khiển view states
├── livekit/
│   ├── agent-control-bar/   # Thanh điều khiển (mic, camera, chat)
│   ├── chat-entry.tsx       # Entry chat message
│   ├── scroll-area/         # Custom scroll với auto-scroll
│   └── ...UI components
```

### Tính năng chính

| Tính năng | Mô tả |
|-----------|-------|
| **Voice Interaction** | Giao tiếp giọng nói real-time với AI Agent |
| **Camera Streaming** | Stream video từ camera để AI mô tả |
| **Screen Sharing** | Chia sẻ màn hình |
| **Chat Transcript** | Hiển thị bản ghi hội thoại (transcription) |
| **Dark/Light Theme** | Hỗ trợ chuyển đổi theme |
| **Pre-connect Buffer** | Hiển thị tin nhắn trước khi kết nối |

### Cấu hình ứng dụng (`app-config.ts`)
```typescript
{
  companyName: 'BlindChat',
  pageTitle: 'BlindChat Voice Agent',
  pageDescription: 'A voice agent support blind users.',
  
  supportsChatInput: true,      // Hỗ trợ nhập chat
  supportsVideoInput: true,     // Hỗ trợ video input
  supportsScreenShare: true,    // Hỗ trợ chia sẻ màn hình
  isPreConnectBufferEnabled: true,
}
```

---

## Luồng hoạt động

### 1. Khởi tạo phiên
```
User → Frontend → POST /api/connection-details → LiveKit Token
                                                      ↓
User ← Frontend ← Connect to LiveKit Room ← AI Agent joins
```

### 2. Voice Interaction
```
User speaks → Frontend captures audio → LiveKit Room → AI Agent
                                                           ↓
                                                    Azure STT (Speech→Text)
                                                           ↓
                                                    OpenAI GPT-4o-mini
                                                           ↓
                                                    Azure TTS (Text→Speech)
                                                           ↓
User hears response ← Frontend plays audio ← LiveKit Room ←
```

### 3. Lưu lịch sử hội thoại
```
AI Agent → ConversationCache (memory buffer)
                    ↓ (sau 5 cặp tin nhắn)
            POST /api/messages → Backend API → SQL Server
```

---

## Cài đặt & Chạy

### Backend
```powershell
cd backend/api
dotnet restore
dotnet ef database update
dotnet watch run
```

### Frontend
```bash
cd frontend
pnpm install
pnpm dev
```

### AI Agent
```bash
cd ai-agent
pip install -r requirements.txt
# Cấu hình .env với OPENAI_API_KEY, AZURE_SPEECH_KEY, LIVEKIT credentials
python agent.py
```

---

## Biến môi trường cần thiết

### AI Agent (`.env`)
```env
OPENAI_API_KEY=xxx
AZURE_SPEECH_KEY=xxx
AZURE_SPEECH_REGION=xxx
BACKEND_BASE_URL=http://localhost:5000
AGENT_LOGIN_USERNAME=agent_user
```

### Frontend (`.env.local`)
```env
LIVEKIT_API_KEY=xxx
LIVEKIT_API_SECRET=xxx
LIVEKIT_URL=wss://your-livekit-url
```

### Backend (`appsettings.json`)
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=localhost\\SQLEXPRESS;Initial Catalog=HmiDb;..."
  }
}
```

---

## Tóm tắt

**BlindChat** là một giải pháp accessibility toàn diện giúp người khiếm thị:
- 🎤 Tương tác bằng giọng nói với AI thông minh
- 📄 Đọc và tóm tắt tài liệu PDF
- 📷 Mô tả những gì camera nhìn thấy
- 🎛️ Điều khiển giao diện bằng giọng nói
- 💬 Duy trì lịch sử hội thoại để cung cấp context liên tục

Ứng dụng được xây dựng với kiến trúc microservices hiện đại, sử dụng LiveKit cho real-time communication và tích hợp các dịch vụ AI hàng đầu từ OpenAI và Azure.

