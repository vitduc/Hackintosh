# Hướng dẫn chuyển hệ điều hành Windows từ HDD sang SSD

Tóm tắt chi tiết quy trình chuyển hệ điều hành Windows từ ổ cứng HDD sang SSD cực đơn giản mà không cần cài lại máy bằng cách sử dụng phần mềm **MiniTool Partition Wizard** (Theo *Thegioididong.com*).

---

## 1. Chuẩn bị trước khi thực hiện

Để quá trình chuyển đổi diễn ra nhanh chóng và an toàn, bạn cần thực hiện hai việc sau:

* **Dọn dẹp máy tính:** Xóa bỏ các dữ liệu rác, tệp tin tạm thời không cần thiết ở hệ điều hành cũ nhằm giảm dung lượng cần di chuyển và tiết kiệm thời gian.
* **Sao lưu dữ liệu:** Tiến hành sao lưu các dữ liệu quan trọng trên ổ cứng HDD trước khi làm để phòng ngừa các sự cố mất dữ liệu ngoài ý muốn.

---

## 2. Các bước chuyển Windows từ HDD sang SSD

### Bước 1: Khởi động tính năng di chuyển
Mở phần mềm **Partition Wizard**, tại giao diện chính tìm và nhấn vào mục **Migrate OS to SSD/HD**.

### Bước 2: Chọn phương thức di chuyển
Hệ thống sẽ hiển thị 2 lựa chọn:
* **Lựa chọn A:** Di chuyển toàn bộ dữ liệu và tất cả các phân vùng từ ổ cứng cũ sang ổ cứng mới.
* **Lựa chọn B:** Chỉ di chuyển riêng phân vùng chứa hệ điều hành Windows sang ổ cứng mới (thường dùng khi bạn muốn lắp và chạy song song cả ổ HDD cũ và ổ SSD mới).

### Bước 3: Cấu hình và tiến hành sao chép

#### Trường hợp 1: Nếu chọn Lựa chọn A
1. Chọn **A** rồi nhấn **Next**.
2. Chọn ổ cứng đích (**SSD mới**) mà bạn muốn lưu trữ dữ liệu rồi nhấn **Next**.
3. Chọn một trong hai chế độ định dạng:
   * *Fit partitions to entire disk:* Tự động điều chỉnh kích thước các phân vùng cho vừa vặn với toàn bộ ổ cứng mới.
   * *Copy Partitions without resize:* Giữ nguyên kích thước gốc của các phân vùng trên ổ cứng mới.
4. Nhấn **Next** -> Nhấn **Finish**.
5. Chọn tiếp **Apply** trên thanh công cụ và nhấn **Yes** để lưu thay đổi và bắt đầu quá trình.

#### Trường hợp 2: Nếu chọn Lựa chọn B
1. Chọn **B** rồi nhấn **Next**.
2. Chọn ổ cứng đích (**SSD mới**) rồi nhấn **Next**.
3. Chọn chế độ định dạng phân vùng tương tự như trên (*Fit partitions to entire disk* hoặc *Copy Partitions without resize*).
4. Nhấn **Next** -> Nhấn **Finish**.
5. Nhấn **Apply** và chọn **Yes** để hệ thống tiến hành dịch chuyển phân vùng hệ điều hành.

---

## 3. Các lưu ý quan trọng để tối ưu hiệu suất

* **Tối ưu hóa SSD:** Trong quá trình thiết lập, bạn nên tích chọn **Align partitions to 1MB** để giúp tối ưu cấu trúc phân vùng và cải thiện hiệu suất hoạt động của ổ cứng SSD sau khi chuyển.
* **Ổ cứng dung lượng lớn:** Nếu ổ cứng mới của bạn có dung lượng trên 2TB, hãy tích chọn **Use GUID Partition Table for the target disk** để hệ thống có thể nhận diện và sử dụng tối đa dung lượng ổ đĩa.
* **Kiểm tra kỹ thông tin:** Dung lượng và tên hiển thị của các ổ cứng trên mỗi máy tính sẽ khác nhau, cần kiểm tra thật kỹ ổ nguồn và ổ đích trước khi bấm *Apply* để tránh ghi đè nhầm dữ liệu.
