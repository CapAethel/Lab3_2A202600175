# Individual Report: Lab 3 - Chatbot vs ReAct Agent

- **Student Name**: [Nguyễn Mai Phương]
- **Student ID**: [2A202600175]
- **Date**: [06/04/2026]

---

## I. Technical Contribution (15 Points)

*Describe your specific contribution to the codebase (e.g., implemented a specific tool, fixed the parser, etc.).*

- **Modules Implemented**: `src/agent/ui.py`, `src/tools/find_common_free_slots.py`
- **Code Highlights**: Triển khai giao diện Streamlit cho agent và tích hợp tool `find_common_free_slots` để tìm khung giờ trống chung.
- **Documentation**: Mô tả cách UI kết nối với ReAct loop, hiển thị lịch sử chat, và cách tool được gọi khi agent cần tìm thời gian rảnh chung.

---

## II. Debugging Case Study (10 Points)

*Phân tích một sự cố cụ thể đã xảy ra trong quá trình làm lab dựa trên nhật ký (logs).* 

- **Mô tả vấn đề**: Hallucinated Chain – send_invitation_email với booking_id sai
- **Chẩn đoán**: LLM đã thực hiện "chain hallucination" – thay vì chờ Observation thực từ tool, mô hình tự mô phỏng toàn bộ chuỗi còn lại trong một lần sinh output. Kết quả là booking_id=12345 (số nguyên giả) được sinh ra trước khi tool book_meeting thực sự trả về meeting_4.
- **Giải pháp**: Thêm rõ trong system prompt dòng hướng dẫn sau: “Generate ONLY ONE Action per step. Do NOT simulate future Observations. Wait for the actual tool result before proceeding."Cập nhật 'book_meeting' để kiểm tra giá trị ngày hợp lệ trước khi gọi tool, đảm bảo 'date >= today'. Nếu ngày không hợp lệ, tool phải trả về lỗi rõ ràng thay vì tiếp tục với dữ liệu sai.

## III. Personal Insights: Chatbot vs ReAct (10 Points)

*Nhìn nhận khác biệt về khả năng suy luận giữa Chatbot thông thường và ReAct agent.*

1.  **Reasoning**: Khung `Thought` giúp agent tách biệt các bước suy luận, chọn công cụ và xử lý kết quả trước khi đưa ra câu trả lời cuối cùng. Thay vì trả lời trực tiếp, agent có thể cân nhắc với `Thought` và quyết định `Action` một cách minh bạch hơn. 
2.  **Reliability**: Agent thỉnh thoảng hoạt động kém hơn Chatbot khi môi trường bị cấu hình sai hoặc tool không sẵn sàng. Trong trường hợp của tôi, ReAct bị chặn bởi lỗi API key và cấu hình local gateway, còn nếu chỉ dùng Chatbot thuần túy thì nó có thể trả lời lỗi một cách đơn giản nhưng không thực thi công cụ. 
3.  **Observation**: Phản hồi từ môi trường (`Observation`) rất quan trọng để agent điều chỉnh bước tiếp theo. Trong logs, sau khi tool gọi thất bại agent nhận `Observation` lỗi và cần tái khởi tạo lại cấu hình hoặc sử dụng biến môi trường đúng, do đó quá trình `Thought-Action-Observation` giúp debug tốt hơn. 

---

## IV. Future Improvements (5 Points)

*How would you scale this for a production-level AI agent system?*

- **Scalability**: Triển khai hàng đợi bất đồng bộ (asynchronous queue) cho các yêu cầu gọi tool để tránh nghẽn, đồng thời tách riêng luồng xử lý tool và luồng sinh ngôn ngữ.
- **Safety**: Thêm một lớp giám sát (supervisor LLM) kiểm tra mọi `Action` trước khi thực thi và phát hiện hallucinative output. Kết hợp rule-based validation và secret scanning để ngăn việc gửi dữ liệu nhạy cảm vào lịch sử commit.
- **Performance**: Dùng cơ sở dữ liệu vector cho tìm kiếm công cụ/phân loại intent, đồng thời cache kết quả tool phổ biến và reuse trả lời khi truy vấn tương tự.

---

> [!NOTE]
> Submit this report by renaming it to `REPORT_[YOUR_NAME].md` and placing it in this folder.
