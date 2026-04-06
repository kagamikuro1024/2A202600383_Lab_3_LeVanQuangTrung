# Test Results Summary
Generated: 2026-04-06 17:13:29
Test File: comprehensive_test_20260406_162056.json

## ✅ Overall Success Rates

| System | Success Rate |
|--------|--------------|
| ChatbotBaseline | 100% |
| Agent v1 | 80% |
| Agent v2 | 100% |

## ⏱️ Latency Metrics

| System | Total | Average | Per Query |
|--------|-------|---------|-----------|
| chatbot | 29877ms | 5975ms | [6910, 1922, 6477, 12836, 1732] |
| agent_v1 | 18831ms | 4708ms | [5631, 3872, 7498, 1830] |
| agent_v2 | 17314ms | 3463ms | [5560, 3245, 3281, 3457, 1771] |

## 📝 Token Usage

| System | Total Tokens | Average | Tokens/Query |
|--------|--------------|---------|--------------|
| chatbot | 3,297 | 659 | 659 |
| agent_v1 | 6,836 | 1709 | 1709 |
| agent_v2 | 9,685 | 1937 | 1937 |

## 🎯 Q2 Accuracy (Grade Penalty) - KEY TEST

This question is the key differentiator:
**Question:** "Tôi nộp bài muộn 2 ngày. Bài của tôi sẽ bị trừ bao nhiêu điểm?"
**Expected Answer:** 20% penalty

| System | Result | Notes |
|--------|--------|-------|
| chatbot | ❌ Wrong | No data access |
| agent_v1 | ✅ Correct | Tool access |
| agent_v2 | ✅ Correct | Tool access |

## 🏆 Conclusion

- **Chatbot** can explain concepts but lacks data access
- **Agent v1** has parsing issues that hurt accuracy
- **Agent v2** improves parsing and adds more tools for better coverage

The key finding is **Q2 accuracy**: Only agents with tool access can answer data-driven questions correctly.

## 📁 Details

See `logs/comprehensive_test_20260406_162056.json` for full test data.
