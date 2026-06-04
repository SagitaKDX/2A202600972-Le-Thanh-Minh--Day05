# CASE STUDY WORKSHOP: MỔ APP AI THẬT
**Sản phẩm phân tích:** MoMo — Trợ thủ AI Moni

---

### 1. Chọn một sản phẩm để dùng thử
* **Sản phẩm:** MoMo — Trợ thủ AI Moni
* **AI feature:** Trợ thủ tài chính, phân tích chi tiêu, chatbot tìm kiếm ưu đãi.
* **Cách truy cập:** App MoMo -> Chatbot Moni.

---

### 2. Dùng thử: Promise vs Reality

* **Product hứa gì?** Giúp người dùng tìm kiếm, tra cứu và sử dụng các ưu đãi, mã giảm giá trên MoMo một cách nhanh chóng, cá nhân hóa thông qua giao tiếp bằng ngôn ngữ tự nhiên.
* **User nào được hứa sẽ được giúp?** Người dùng MoMo đang có sẵn điểm MoMo Rewards và muốn đổi điểm lấy các voucher chi tiêu (ví dụ: xem phim, mua cafe) phù hợp với nhu cầu cá nhân.
* **Bạn kỳ vọng AI làm được task nào?** Khi đưa ra câu lệnh tìm kiếm mã giảm giá kèm điều kiện cụ thể (dưới 10.000 điểm, dùng để xem phim), AI sẽ tự động lọc, truy xuất dữ liệu và hiển thị danh sách voucher dưới dạng các thẻ tương tác trực quan (Interactive UI Cards/Carousel) có sẵn nút "Đổi ngay", tương tự như khi hỏi chung chung.
* **Khi dùng thật, điểm gãy xuất hiện ở đâu?**
    * **Evidence 1 (Prompt chung chung):** Khi gõ *"tất cả mã giảm giá dưới 10 nghìn momo rewards can you find that for me"*, AI hoạt động hoàn hảo, trả về giao diện dạng Carousel chứa các thẻ quà tặng (Highlands Coffee, Chuyển tiền MoMo) kèm số điểm rõ ràng, có thể click vào để tương tác.
    * **Evidence 2 (Prompt có điều kiện ngách - Điểm gãy):** Khi bổ sung thêm mục đích sử dụng *"vho việc xem phim"* (gõ nhầm "cho" thành "vho"), AI vẫn hiểu đúng ý nghĩa dữ liệu (Intent) và lấy ra được voucher xem phim trị giá 10K với giá đổi là 9.999 điểm (Hệ thống RAG chạy đúng dữ liệu). Tuy nhiên, thay vì render ra giao diện thẻ quà tặng dạng Carousel như luồng trên, hệ thống lại **fallback về hiển thị một bảng dữ liệu text/markdown thuần (Static Table)**. Người dùng chỉ đọc được thông tin chứ **không thể nhấn vào để đổi voucher** trực tiếp từ màn hình chat.

---

### 3. Vẽ 4 paths

| Path | Câu hỏi cần trả lời | Thực tế trong Product (Moni) |
| :--- | :--- | :--- |
| **Happy** | Khi AI đúng và tự tin, user thấy gì? | User gõ prompt chung chung -> AI nhận diện đúng Intent, gọi đúng API Tool Calling -> Hiển thị danh sách Voucher dạng Carousel/Card sinh động, có nút tương tác (Bấm để đổi thưởng ngay). |
| **Low-confidence** | Khi AI không chắc, hệ thống có hỏi lại, show options hoặc chuyển người không? | *Path này hiện tại bị khuyết.* Khi user thêm bộ lọc "xem phim" (hoặc gõ sai chính tả), AI không chắc chắn cách map data vào UI component nào nên tự động chuyển sang hiển thị dạng Table thô thay vì hiển thị các option hoặc hỏi lại: *"Moni tìm thấy 1 ưu đãi xem phim, bạn có muốn đi tới trang đổi thưởng ngay không?"* |
| **Failure** | Khi AI sai, user biết bằng cách nào và sửa thế nào? | AI sai ở tầng **Format UI Output** (hiển thị bảng text thay vì thẻ tương tác). User nhận ra ngay vì giao diện bị đổi từ Card sang Table thô ráp và bị mất các nút hành động. User buộc phải tự sửa bằng cách: Đổi prompt khác ít điều kiện ngách hơn, hoặc thoát chatbot để vào mục "Ưu đãi" của MoMo tìm kiếm thủ công. |
| **Correction** | Khi user sửa, correction có được lưu/log/học lại không hay biến mất? | Hệ thống xử lý độc lập theo từng lượt chat (Stateless ở tầng UI). Việc user xoá chữ "xem phim" để bắt Moni hiện lại dạng Card chỉ là giải pháp đường vòng tạm thời của user, AI không tự động ghi nhận lỗi render này để tự học hay cải tiến luồng mapping UI cho lần sau. |

---

### 4. Viết finding thành quyết định

* **Finding:** Khi user **cung cấp thêm điều kiện ngách hoặc bộ lọc chi tiết (như danh mục "cho việc xem phim") vào câu lệnh tìm voucher**, 
    AI/product **hiểu đúng intent và dữ liệu nhưng bị lỗi nhận diện hoặc không trigger được API vẽ Interactive Component (Tool Calling), dẫn đến việc fallback về render text dạng bảng (Markdown Table)**, 
    hậu quả là **luồng trải nghiệm bị đứt gãy (friction), user có thông tin dữ liệu nhưng không thể thực hiện hành động tiếp theo (no Call-to-Action/mất nút đổi), làm giảm tỷ lệ chuyển đổi (Conversion Rate) của tính năng đổi thưởng.**
* **Lỗi thuộc layer:** Intent-to-UI Mapping / UX Recovery.
* **Nên sửa bằng:** * *UX/Requirement:* Định nghĩa quy tắc cứng (Hard-rule) cho LLM: Bất cứ khi nào dữ liệu truy xuất trả về (Object) thuộc danh mục `Voucher / Mã giảm giá`, kết quả hiển thị bắt buộc phải được map vào UI Component dạng Card/Carousel hoặc List có nút bấm tương tác. Tuyệt đối không sử dụng định dạng bảng text (Markdown Table) cho thực thể có tính chất giao dịch.
    * *Fallback UX:* Trong trường hợp hệ thống thiếu Metadata (như hình ảnh, ID deep-link) không đủ để dựng Card hoàn chỉnh, câu trả lời dạng bảng text bắt buộc phải chèn thêm một nút bấm hoặc hyperlink động ở cuối bảng: `[Bấm vào đây để đi đến trang Đổi voucher Xem Phim]`.

---

### 5. Sketch as-is / to-be
