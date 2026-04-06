# Individual Report: Lab 3 - Chatbot vs ReAct Agent

- **Student Name**: Trinh Uyen Chi
- **Student ID**: 2A202600435
- **Date**: 2026-04-06

---

## I. Technical Contribution (15 Points)


- **Modules Implementated**: `src/tools/search_center.py`, `app.py`
- **Code Highlights**:
```
print(f"  [Tool Execution] Đang dùng Google Search: '{query}'...")
    
    api_key = os.environ.get("SERPER_API_KEY")
    if not api_key:
        return json.dumps({"status": "error", "message": "Thiếu SERPER_API_KEY"})

    url = "https://google.serper.dev/search"
    
    # Cấu hình tìm kiếm: gl=vn (Việt Nam), hl=vi (Tiếng Việt)
    payload = json.dumps({
        "q": query,
        "gl": "vn",
        "hl": "vi",
        "num": 5 # Lấy 5 kết quả
    })
    headers = {
        'X-API-KEY': api_key,
        'Content-Type': 'application/json'
    }
```
```
def search_course_info(query: str) -> str:
    """
    HÃY SỬ DỤNG công cụ này khi cần tra cứu thông tin thực tế về khóa học (học phí, địa điểm, lịch học...).
    
    Lưu ý về tham số 'query':
    - Phải là từ khóa tìm kiếm ngắn gọn, trúng đích.
    
    Đầu ra (Output) sẽ là một chuỗi JSON có cấu trúc. Bạn cần phân tích cú pháp JSON này 
    để trích xuất các thông tin cần thiết như tên trung tâm, học phí, và thời lượng.
    """
    print(f"  [Tool Execution] Đang dùng Google Search: '{query}'...")
    
    api_key = os.environ.get("SERPER_API_KEY")
    if not api_key:
        return json.dumps({"status": "error", "message": "Thiếu SERPER_API_KEY"})
```
- **Documentation**: 
I created a tool named search_center. Its main function is searching online for the prompt the user requested. This tool is essential for a real-time chatbot that needs up-to-date information. The tool help the agent provide the user with the exact location, available courses, total time for the courses and the price of each course. Moreover, I created a simple UI that allows users to have a continuous conversation with the AI Agent. 

---

## II. Debugging Case Study (10 Points)

- **Problem Description**: The agent hallucinate the information.
- **Log Source**: 
```
**Trung tâm 1: Trung Tâm Tin Học**
- Tổng học phí: 3,500,000 VNĐ
- Tổng số buổi: 10 buổi
- Thời lượng mỗi buổi (ước tính): 2.5 giờ
- Tổng thời lượng học: 10 * 2.5 = 25 giờ
- Chi phí/giờ: 3,500,000 / 25

**Trung tâm 2: TopAI (Khoahocai.net)**
- Tổng học phí: 3,500,000 VNĐ
- Tổng số buổi: 10 buổi
- Thời lượng mỗi buổi (ước tính): 2.5 giờ
- Tổng thời lượng học: 10 * 2.5 = 25 giờ
- Chi phí/giờ: 3,500,000 / 25
```
- **Diagnosis**:

The LLM hallucinated because of a lack of specific data in the tool's response and a weak system prompt.

In my opinion, the search_center tool likely provided the names of the centers but failed to provide the exact price or number of sessions for those specific courses. 

When the LLM didn't find the real prices in the Observation, it didn't want to fail. Instead, it created a plausible-looking structure. It created a template for Center 1 and simply copied-pasted the exact same fake numbers to Center 2 to make the comparison look balanced.

Moreover, the original prompt asked for a specific math calculation (VND/hour). To satisfy this request without real data, the model guessed the numbers to complete the math.
- **Solution**: 

I fixed this by improving the System Prompt and adding a Constraint to the agent's logic. I updated the get_system_prompt() in agent.py to include a rule: "If the Observation does not contain a specific price or duration, DO NOT invent numbers. State that the information is unavailable."

---

## III. Personal Insights: Chatbot vs ReAct (10 Points)

1.  **Reasoning**: How did the `Thought` block help the agent compared to a direct Chatbot answer?

In a normal chatbot, the AI just guesses the next word. In a ReAct agent, the Thought block acts like a brainstorming phase before the AI speaks. It helps the AI plan out what it needs to do first.

By thinking first, the AI realizes it doesn't have all the answers yet and decides to use a tool to get real facts instead of making things up.

2.  **Reliability**: In which cases did the Agent actually perform *worse* than the Chatbot?

Even though the ReAct Agent is smarter than AI chatbot, it isn't always better. It performed worse when:

*The question was too simple*: If I just say "Hi!" and the agent still tries to think and act, it will be a waste of time and makes the response slow.

*It got stuck in a loop*: Sometimes the tool gave a confusing result, and the agent just kept trying the same wrong thing over and over until it hit the 5-step limit.

3.  **Observation**: How did the environment feedback (observations) influence the next steps?

It tells the AI if its plan worked. If the observation says "Không tìm thấy khóa học nào" the AI sees that and changes its next "Thought" to try a different search.

It keeps the AI being honest because it has to use the information it just observed to give the final answer.

---

## IV. Future Improvements (5 Points)

*How would you scale this for a production-level AI agent system?*

- **Scalability**: Right now, the code waits for one tool to finish before doing anything else. I would make it "Asynchronous" which means multi-tasking. This means the AI could talk to 100 students at the same time without making anyone wait in line.
- **Safety**: I would add a "Supervisor" AI. The supervisor would check the agent’s actions to make sure it isn't giving wrong advice or doing something dangerous with the data.
- **Performance**: If I had 1,000 tools, the AI's "instructions" would become too long and expensive. I would use a Vector Database, which acts as a library index that only shows the AI the 3 or 4 tools it actually needs for that specific question, keeping the system fast and cheap.
---
