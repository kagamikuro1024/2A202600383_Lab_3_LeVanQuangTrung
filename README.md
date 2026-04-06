# 🤖 Lab 3: Chatbot vs ReAct Agent - học viên khoá Agentic AI

> **Mục tiêu:** So sánh một chatbot LLM đơn giản với một agent ReAct thông minh có hệ thống gọi tools. Hiểu rõ khi nào nên dùng chatbot, khi nào cần agent.

## 📋 Tổng quan dự án

Dự án này là một **production-ready lab** để hiểu:
- **Chatbot Baseline**: Sử dụng LLM thuần (không tools) - tốt cho Q&A nhưng không giải quyết được vấn đề multi-step
- **Agent v1**: Triển khai vòng lặp ReAct cơ bản (Thought → Action → Observation) - đầu tiên thử thách với 3 tools
- **Agent v2**: Cải tiến v1 với JSON parsing tốt hơn, thêm 5 tools, và prompt tối ưu

### 🎯 Key Insight
Chatbot **có thể giải thích** kiến thức C (pointer, recursion, etc.), nhưng **không thể trả lời** câu hỏi yêu cầu truy cập dữ liệu (deadline môn học, điểm trừ muộn nộp). **Đó là nơi agent tỏa sáng!**

---

## 📊 So sánh 3 phiên bản

| Tiêu chí | Chatbot | Agent v1 | Agent v2 |
|----------|---------|----------|----------|
| **Tools** | 0 | 3 | 5 |
| **Multi-step** | ❌ | ✅ | ✅ |
| **JSON Parsing** | N/A | Cơ bản | Robust (xử lý markdown JSON) |
| **Q2 Accuracy* test | ❌ 0% | ✅ 100% | ✅ 100% |
| **Tokens/query** | 659 | 1709 | 1937 |
| **Latency** | 5975ms | 4708ms | 3463ms |

*Q2 = "Nộp muộn 2 ngày thì trừ bao nhiêu điểm?" (câu hỏi yêu cầu dữ liệu quy định)

---

## 🚀 Cài đặt & Chạy nhanh

### 1. Clone & Setup
```bash
# Clone repo
cd Day-3-Lab-Chatbot-vs-react-agent

# Copy .env và điền API key
cp .env.example .env
# Mở .env và điền: OPENAI_API_KEY=sk-...
```

### 2. Cài dependencies
```bash
pip install -r requirements.txt
```

### 3. Chạy test toàn bộ (recommended đầu tiên)
```bash
python test_all_versions.py
```
Lệnh này sẽ:
- Khởi động 3 phiên bản (Chatbot, Agent v1, Agent v2)
- Chạy 5 scenarios tiếng Việt về C programming
- Ghi kết quả vào `logs/comprehensive_test_*.json`
- In bảng so sánh trực tiếp

Sau khi chạy xong, generate báo cáo summary:
```bash
python summarize_results.py
```

### 4. Chạy individual versions
**Chatbot Baseline:**
```python
from src.agent.chatbot import ChatbotBaseline
bot = ChatbotBaseline(provider="openai")
response = bot.chat("Con trỏ (pointer) là gì? Cho ví dụ thực tế.")
print(response)
```

**Agent v1:**
```python
from src.agent.agent import ReActAgent
agent = ReActAgent(provider="openai", max_steps=10)
result = agent.run("Tôi nộp bài muộn 2 ngày. Bài của tôi sẽ bị trừ bao nhiêu điểm?")
print(result)
```

**Agent v2 (improved):**
```python
from src.agent.agent_v2 import ReActAgentV2
agent = ReActAgentV2(provider="openai", max_steps=5)
result = agent.run("Cho tôi roadmap học recursion từ cơ bản đến nâng cao.")
print(result)
```

---

## 📁 Cấu trúc thư mục

```
├── src/
│   ├── agent/
│   │   ├── chatbot.py         ← Baseline (không tools)
│   │   ├── agent.py           ← Agent v1 (3 tools)
│   │   └── agent_v2.py        ← Agent v2 (5 tools - LỰA CHỌN CHỢ)
│   ├── core/
│   │   ├── llm_provider.py    ← Interface
│   │   ├── openai_provider.py
│   │   ├── gemini_provider.py
│   │   └── local_provider.py
│   ├── tools/
│   │   ├── base_tool.py
│   │   └── teaching_assistant_tools.py  ← 5 tools cho trợ giảng
│   └── telemetry/
│       ├── logger.py
│       └── metrics.py
├── data/                      ← JSON data (tài liệu, quy định, etc)
│   ├── tai_lieu_hoc_tap.json
│   ├── quy_dinh_mon_hoc.json
│   └── ...
├── logs/                      ← Kết quả test (JSON traces)
├── report/
│   ├── group_report/
│   └── individual_reports/
├── test_all_versions.py       ← Test suite chính
├── summarize_results.py       ← Generate báo cáo
└── SCORING.md                 ← Rubric chấm điểm
```

---

## 🏠 Dùng Local Models (không cần API key)

Nếu không muốn dùng OpenAI/Gemini, chạy Phi-3 trực tiếp trên CPU:

### 1. Download model Phi-3 (2.2GB)
```bash
# Download từ Hugging Face
wget https://huggingface.co/microsoft/Phi-3-mini-4k-instruct-gguf/resolve/main/Phi-3-mini-4k-instruct-q4.gguf

# Hoặc dùng Browser: https://huggingface.co/microsoft/Phi-3-mini-4k-instruct-gguf
```

### 2. Tạo folder `models/` và move file
```bash
mkdir models
mv Phi-3-mini-4k-instruct-q4.gguf models/
```

### 3. Update `.env`
```env
DEFAULT_PROVIDER=local
LOCAL_MODEL_PATH=./models/Phi-3-mini-4k-instruct-q4.gguf
```

### 4. Chạy lại tests
```bash
python test_all_versions.py
```

---

## 🔍 5 Test Scenarios (Tiếng Việt)

Dự án test 5 câu hỏi thực tế sinh viên hỏi:

1. **Q1: Pointers** - "Tôi không hiểu con trỏ hoạt động như thế nào. Cho ví dụ thực tế."
   - ✅ Cả 3 đều làm tốt (là Q&A khái niệm)

2. **Q2: Grade Penalty** ⭐ - "Tôi nộp bài muộn 2 ngày. Bài của tôi sẽ bị trừ bao nhiêu điểm?"
   - ❌ Chatbot: không biết, phải hallucinate
   - ✅ Agent v1/v2: gọi `get_course_policy()` + `calculate_grade_penalty()` → **20% đúng**

3. **Q3: Buffer Overflow** - "Làm sao tránh buffer overflow khi dùng strings?"
   - ✅ Chatbot & Agents đều OKay

4. **Q4: Recursion Roadmap** - "Cho tôi roadmap học recursion từ cơ bản → nâng cao."
   - ✅ Agent v2 dùng `create_learning_roadmap()` tốt hơn

5. **Q5: Deadlines** - "Tôi có 3 bài chưa nộp. Deadline bao giờ? Còn bao lâu?"
   - ✅ Agent v1/v2 gọi `get_course_policy()` → lấy dữ liệu thực

---

## 🛠️ 5 Tools có sẵn

Tất cả tự động load từ các JSON data files:

1. **`search_learning_material`** - Tìm tài liệu học tập (pointer, loop, string, etc)
2. **`get_course_policy`** - Lấy quy định môn (deadline, scoring, late submission)
3. **`calculate_grade_penalty`** - Tính điểm trừ muộn nộp
4. **`create_code_example`** - Sinh code example cho topic
5. **`create_learning_roadmap`** - Tạo roadmap học từ cơ bản → nâng cao

Xem chi tiết: [src/tools/teaching_assistant_tools.py](src/tools/teaching_assistant_tools.py)

---

## 📊 Phân tích kết quả

Sau khi chạy `test_all_versions.py`:

### Xem logs chi tiết
```bash
# Lấy JSON trace mới nhất
cat logs/comprehensive_test_*.json | python -m json.tool

# Hoặc xem parse errors
grep "parse_error" logs/comprehensive_test_*.json
```

### Generate báo cáo summary
```bash
python summarize_results.py
```
Sẽ tạo `TEST_RESULTS_SUMMARY.md` với:
- Success rates
- Latency comparison
- Token usage
- Q2 accuracy test
- Scenario breakdown

### Key Metrics theo EVALUATION.md

1. **Token Efficiency** - Ít token = ít chi phí
2. **Latency** - Tốc độ response (< 2s là tốt)
3. **Loop count** - Số bước ReAct cần (< 5 là ideal)
4. **Failure Analysis** - Phân loại lỗi: JSON parser, hallucination, timeout

---

## 🎓 How to Learn

### 1. **Hiểu vấn đề** (15 phút)
```bash
# Chạy test toàn bộ
python test_all_versions.py

# Đọc results
python summarize_results.py
cat TEST_RESULTS_SUMMARY.md
```

### 2. **Tìm hiểu cây số** (30 phút)
- Mở `logs/comprehensive_test_*.json`
- Tìm một trace FAILED (agent vừa fail)
- Đọc `thinking_steps` để thấy agent suy nghĩ gì
- Tìm `parse_errors` để thấy nó parse sai chỗ nào

**Ví dụ:**
```json
{
  "success": false,
  "parse_errors": 1,
  "error": "Could not extract JSON action",
  "thinking_steps": [...],
  "observations": [...]
}
```

### 3. **So sánh & Improve** (45 phút)
- So sánh **v1 vs v2** - xem v2 fix cái gì
- Nhìn prompt trong `agent_v2.py` line X
- Đề xuất cải tiến: tool description, system prompt, retry logic

### 4. **Code & Test** (theo yêu cầu)
- Implement Agent v3 (nếu cần)
- Add thêm tool
- Run test lại, capture kết quả

---

## 📝 Viết báo cáo (theo SCORING.md)

### Group Report
File: `report/group_report/GROUP_REPORT_LAB3.md`
- Thảo luận tại sao Chatbot vs Agent khác nhau
- Phân tích Q2 accuracy test
- Flowchart ReAct logic
- Suggestions để improve

### Individual Report
File: `report/individual_reports/REPORT_[TEN].md`
- Code modules bạn implement
- 1 debugging case study (trace của lỗi + cách fix)
- Insights cá nhân
- Ideas tương lai (RAG, multi-agent)

---

## ⚡ Troubleshooting

### ❌ `OPENAI_API_KEY not found`
```bash
# Kiểm tra .env
cat .env

# Hoặc set trực tiếp
export OPENAI_API_KEY=sk-...
python test_all_versions.py
```

### ❌ `ModuleNotFoundError: src`
```bash
# Chạy từ root folder
cd Day-3-Lab-Chatbot-vs-react-agent
python test_all_versions.py
```

### ❌ `JSON parse error`
- Đây là điểm khác biệt v1 vs v2!
- v1 có vấn đề parse JSON wrapped in markdown
- v2 có `_extract_json_safely()` để fix
- Xem: [agent_v2.py](src/agent/agent_v2.py#L120)

### ❌ Agent bị stuck (infinite loop)
- Check `max_steps` limit
- Xem `thinking_steps` để agent suy nghĩ gì
- Có thể prompt chưa rõ hoặc tools chưa phù hợp

---

## 📚 References

- **ReAct Paper**: [Reasoning + Acting](https://arxiv.org/abs/2210.03629)
- **Evaluation**: [EVALUATION.md](EVALUATION.md) - Chi tiết 4 metrics
- **Scoring**: [SCORING.md](SCORING.md) - Rubric chấm 100 points
- **Instructor Guide**: [INSTRUCTOR_GUIDE.md](INSTRUCTOR_GUIDE.md) - Timeline 4 giờ

---

## 🏆 Expectations

Mục tiêu của lab:
1. ✅ Hiểu rõ **Chatbot vs Agent** (khi nào dùng cái nào?)
2. ✅ Implement **ReAct loop** từ đầu (không copy-paste)
3. ✅ Debug **bằng telemetry** (không guessing)
4. ✅ Improve **v1 → v2** (dữ liệu-driven, không intuition)
5. ✅ Viết báo cáo **data-driven** (có số liệu, không "tớ chữa tốt")

---

*Happy Debugging! Nhớ đọc logs, không hallucinate như model 😄*
