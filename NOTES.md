# Ghi chú phân tích · Day 04

Họ và tên: Lê hữu Toàn
MSSV:2A202603660

## 1. Case được chọn

- **ID case:** A5
- **Câu hỏi:** Yêu cầu danh sách đầy đủ incident GitHub Actions trong năm 2025 và nói rõ độ đầy đủ, không bịa.
- **Điểm lần 1:** 35/100.
- **Câu trả lời lần 1:** Agent nói không có dữ liệu năm 2025, nhưng đưa danh sách incident năm 2026 và nói đã kiểm tra lịch sử.

## 2. Bằng chứng trong trace

`get_status` tìm trong tối đa 50 incident gần nhất; kết quả trả về các incident năm 2026 và có ghi chú rằng không tìm thấy không chứng minh incident không tồn tại. Vì vậy trace không bao phủ toàn bộ năm 2025 và không thể chứng minh có hay không có incident trong năm đó.

Nguyên nhân có thể xảy ra: model cố cung cấp danh sách thay thế từ năm có dữ liệu, dù câu hỏi yêu cầu riêng năm 2025. Việc nói đã kiểm tra lịch sử cũng khiến phạm vi bằng chứng nghe đầy đủ hơn thực tế.

## 3. Thay đổi đã thực hiện

- **Sửa gì, ở đâu:** Mở rộng `SYSTEM_PROMPT` trong `agent.py` để nhắc model phân biệt timestamp snapshot, nêu rõ giới hạn tìm kiếm 50 incident, không suy luận từ kết quả rỗng và chỉ nêu thông tin có trong tool result.
- **Vì sao:** Nhắm đến các lỗi về scope, độ đầy đủ và khẳng định vượt quá bằng chứng.
- **Contract nguồn:** `agent.py` đã kiểm tra câu trả lời có chứa URL incident lấy từ `get_incident_details` thành công hay chưa; nếu thiếu thì ghi `validation_missing_source` và cho model cơ hội sửa trong giới hạn step.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---:|---:|
| Điểm tổng | 79,32/100 | 83,73/100 |
| A5 | 35/100 | 55/100 |
| Số model calls của agent | 53 | 51 |
| Tool calls | 49 | 45 |
| Tổng token agent + judge | 100.581 | 93.880 |
| Chi phí agent + judge (theo usage) | khoảng $0,02144 | khoảng $0,02025 |

- **Cấu hình giữ nguyên:** OpenRouter `openai/gpt-4o-mini` cho agent và judge, `max_steps=8`, dataset `core`.
- **Cải thiện:** Điểm tổng tăng 4,41; A5 tăng 20 điểm. E3, E5, E6, A1, A3, A4, A6, A7, H5 và H7 cũng tăng điểm.
- **Lỗi còn lại:** A5 vẫn thay bằng danh sách năm 2026 và vẫn kết luận về năm 2025 dù dữ liệu chỉ bao phủ 50 incident gần nhất. H7 cũng biết Cloudflare bị timeout nhưng vẫn kết luận Spectrum đang hoạt động bình thường. H2 giảm từ 100 xuống 50.

## 5. Giới hạn

- Điểm là ước lượng từ LLM judge; agent và judge cùng dùng một model nên có thể có thiên lệch.
- Hai lần chạy đều đủ 22/22 case và cùng cấu hình, nhưng model có thể trả lời khác nhau giữa các lần chạy. Một lần mỗi phiên bản không chứng minh cải tiến ổn định.
- Snapshot chỉ phản ánh dữ liệu cố định trong lab; kết quả không xác nhận trạng thái dịch vụ trực tiếp hiện nay.
- Prompt mới giúp điểm tổng tăng nhưng chưa sửa dứt điểm A5; cần tiếp tục cải tiến cách từ chối suy luận vượt quá phạm vi dữ liệu.

## Demo và thảo luận

- **Trace nhiều bước:** Dùng case E2 trong `eval_after.json`: bước 1 gọi `get_status`, bước 2 gọi `get_incident_details` cho các incident liên quan. Chỉ ra tool result được nối lại trước khi model trả lời cuối.
- **Lỗi agent:** Dùng case H7 trong `eval_after.json`: tool GitHub trả trạng thái, tool Cloudflare trả `TimeoutError`, nhưng câu trả lời vẫn nói Spectrum được coi là operational. Timeout chỉ cho biết không kiểm tra được, không phải bằng chứng dịch vụ bình thường.
- **Câu hỏi thảo luận:** Làm sao để agent chỉ báo phần Cloudflare chưa xác minh mà vẫn trả được phần GitHub đã có bằng chứng?
