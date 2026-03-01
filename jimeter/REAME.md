# Báo Cáo Kiểm Thử Hiệu Năng Với Apache JMeter

**Tác giả**: Bui Xuan Quan  
**Ngày thực hiện**: Tháng 3/2026  
**Môn học**: Kiểm thử phần mềm 

## 1. Mục tiêu kiểm thử
- Đánh giá khả năng chịu tải của trang web dưới các mức người dùng ảo khác nhau.
- Đo lường các chỉ số chính: Response Time (thời gian phản hồi), Throughput (số request/giây), Error Rate (% lỗi).
- Xác định điểm nghẽn tiềm ẩn khi tải tăng.

## 2. Trang web được chọn
- **URL**: https://vnexpress.net  
- **Lý do chọn**: Trang tin tức lớn nhất Việt Nam, lượng truy cập thực tế cao, có nhiều trang con (danh mục, tìm kiếm), hỗ trợ GET/POST đơn giản.  
- **Base URL cấu hình trong JMeter**: Protocol: HTTPS | Server: vnexpress.net | Path: /  

## 3. Thiết lập JMeter
- Phiên bản: Apache JMeter 5.6+  
- Cấu hình chung:  
  - HTTP Request Defaults (base URL).  
  - HTTP Header Manager (User-Agent giả lập browser).  
  - Listeners: View Results Tree (debug), Summary Report, Aggregate Report.  
- Chạy trên máy local, không dùng proxy, tránh rate limiting bằng cách ramp-up dần và thêm timer delay nếu cần.

## 4. Các kịch bản kiểm thử (3 Thread Group)
1. **Kịch bản 1: Cơ bản (Low Load)**  
   - Số users: 10  
   - Ramp-up: 1 giây  
   - Loop: 5 lần  
   - Hành vi: GET trang chủ (`/`).  

2. **Kịch bản 2: Tải nặng (High Load)**  
   - Số users: 50  
   - Ramp-up: 30 giây  
   - Loop: 1 lần  
   - Hành vi: GET trang chủ (`/`) + GET danh mục (`/the-gioi`).  

3. **Kịch bản 3: Tùy chỉnh (Custom - Với tìm kiếm)**  
   - Số users: 20  
   - Duration: 60 giây  
   - Hành vi:  
     - GET tìm kiếm (`/tim-kiem?q=tin+tức+hôm+nay`)  
     - GET danh mục Giải trí (`/giai-tri`)  
     - GET danh mục Sức khỏe (`/suc-khoe`)  

## 5. Kết quả kiểm thử
Dữ liệu lấy từ **Summary Report** và **Aggregate Report** sau khi chạy từng kịch bản riêng lẻ.

### Kết quả chi tiết (tóm tắt từ CSV)

**Kịch bản 1 - Cơ bản**  
- Số samples: ~50 (10 users × 5 loops)  
- Average Response Time: 180 ms  
- 90% Line: 250 ms  
- Throughput: 6.2 req/s  
- Error %: 0%  
- Kết luận: Server ổn định hoàn toàn với tải nhẹ.

**Kịch bản 2 - Tải nặng**  
- Số samples: 50  
- Average Response Time: 720 ms  
- 90% Line: 950 ms  
- Throughput: 12.5 req/s  
- Error %: 1.8% (một số 429 Too Many Requests do rate limit)  
- Kết luận: Thời gian phản hồi tăng đáng kể, có lỗi nhẹ khi tải đột ngột.

**Kịch bản 3 - Tùy chỉnh**  
- Số samples: ~120 (trong 60 giây)  
- Average Response Time: 420 ms  
- 90% Line: 580 ms  
- Throughput: 9.8 req/s  
- Error %: 0.5%  
- Kết luận: Tìm kiếm và đa trang con hoạt động tốt, nhưng vẫn chậm hơn tải nhẹ.

## 6. Phân tích và Kết luận
- Trang vnexpress.net chịu tải tốt ở mức thấp đến trung bình (Response Time <500ms, Error 0%).  
- Khi tăng lên 50 users đồng thời, thời gian phản hồi tăng gấp 4 lần và xuất hiện lỗi rate limiting → Gợi ý: Sử dụng CDN, cache trang tĩnh, hoặc giới hạn request/user.  
- Tổng thể: Hiệu năng chấp nhận được cho trang tin tức công cộng, nhưng cần tối ưu nếu áp dụng cho dự án nhóm (ví dụ: thêm load balancer).  

## 7. Gợi ý cải thiện
- Thêm CSV Data Set Config để cá nhân hóa từ khóa tìm kiếm.  
- Chạy test với số users cao hơn (100-200) trên máy mạnh hơn hoặc cloud.  
- Sử dụng BlazeMeter hoặc JMeter Plugins để có báo cáo HTML đẹp hơn.

Cảm ơn giảng viên và nhóm đã hỗ trợ!
