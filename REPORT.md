REPORT: Phân tích Virtual Machine và Container
1. Bảng so sánh VM và Container
  Tiêu chí                Virtual Machine (VM)    Container
  Dung lượng Image        Thường lớn vì mỗi máy   Thường nhỏ hơn vì image
                          ảo cần một hệ điều hành chủ yếu chứa ứng dụng,
                          khách cùng các thư viện thư viện và các thành
                          và ứng dụng. Dung lượng phần cần thiết;
                          có thể từ vài GB trở    container dùng chung
                          lên.                    kernel của hệ điều hành
                                                  máy chủ.
  Thời gian khởi động     Thường lâu hơn do phải  Thường nhanh hơn vì
                          khởi động cả hệ điều    không cần khởi động một
                          hành khách.             hệ điều hành riêng hoàn
                                                  chỉnh.
  Hao phí OS              Cao hơn do mỗi VM có hệ Thấp hơn do các
                          điều hành khách và tiêu container trên cùng máy
                          thụ CPU, RAM, ổ đĩa cho chủ chia sẻ kernel hệ
                          hệ điều hành đó.        điều hành.
  Mức độ cô lập           Thường mạnh hơn vì mỗi  Cô lập ở cấp hệ điều
                          VM có hệ điều hành      hành bằng các cơ chế
                          khách riêng và được ảo  như namespaces và
                          hóa phần cứng.          cgroups; mức cô lập phụ
                                                  thuộc cấu hình và nền
                                                  tảng container.
  Khả năng mở rộng        Có thể mở rộng nhưng    Thường thuận tiện để
                          thường cần nhiều tài    tạo nhiều bản sao
                          nguyên và thời gian để  nhanh, phù hợp với
                          tạo, khởi động VM mới.  triển khai dịch vụ và
                                              môi trường tự động hóa.
Tóm lại, VM phù hợp khi cần ranh giới cô lập mạnh hoặc cần chạy các hệ
điều hành khách khác nhau. Container phù hợp khi muốn đóng gói ứng dụng
nhất quán, triển khai nhanh và sử dụng tài nguyên hiệu quả. Container
không hoàn toàn thay thế VM; hai công nghệ có thể được dùng cùng nhau,
chẳng hạn chạy các container bên trong một máy ảo.
2. Vai trò của Namespaces và cgroups trong Container
Namespaces là cơ chế của Linux giúp tạo ra các góc nhìn tài nguyên
riêng cho từng nhóm tiến trình. Nhờ đó, một tiến trình bên trong
container có thể nhìn thấy môi trường được cô lập thay vì toàn bộ tài
nguyên như tiến trình trên máy chủ.
- PID namespace (Process): cô lập không gian định danh tiến trình.
  Tiến trình trong container có thể thấy tiến trình của container mình
  mà không nhất thiết nhìn thấy toàn bộ tiến trình trên máy chủ.
- Network namespace (Network): cung cấp ngăn xếp mạng riêng, gồm
  giao diện mạng, địa chỉ IP, bảng định tuyến và các thiết lập mạng
  liên quan. Docker có thể kết nối các network namespace qua mạng
  container.
- Mount namespace (Mount points): cô lập các điểm gắn kết hệ thống
  tệp. Container có thể có cách nhìn riêng về thư mục gốc và các mount
  points, giúp ứng dụng làm việc với hệ thống tệp đã được bố trí cho
  nó.
Namespaces chủ yếu giúp cô lập và phân tách góc nhìn tài nguyên, chứ
không tự mình đặt giới hạn mức sử dụng tài nguyên.
Control Groups (cgroups) là cơ chế Linux dùng để theo dõi, phân bổ
và giới hạn tài nguyên mà một nhóm tiến trình sử dụng. Với container,
cgroups có thể được cấu hình để giới hạn hoặc quản lý:
- CPU: đặt hạn mức hoặc tỷ lệ CPU mà nhóm tiến trình được sử dụng,
  giúp một container không chiếm toàn bộ năng lực xử lý.
- RAM: giới hạn lượng bộ nhớ được phép sử dụng; khi vượt ngưỡng,
  hệ thống có thể xử lý theo chính sách bộ nhớ, bao gồm khả năng kết
  thúc tiến trình do thiếu bộ nhớ.
Vì vậy, có thể nhớ ngắn gọn: namespaces giúp cô lập, còn cgroups
giúp quản lý và giới hạn tài nguyên. Hai cơ chế kết hợp giúp container
chia sẻ kernel nhưng vẫn có môi trường làm việc riêng và mức sử dụng tài
nguyên được kiểm soát.
3. Các thông số cơ bản từ lệnh docker info
Dựa trên ảnh chụp đầu ra lệnh docker info, các thông số được ghi nhận
như sau:
  Thông số                         Giá trị
  Server Version                   29.9.0
  Cgroup Driver                    systemd
  Số lượng Containers              2
  Containers đang chạy (Running)   1
  Containers đã dừng (Stopped)     1
  Containers tạm dừng (Paused)     0
  Số lượng Images                  2
Nhận xét: Docker Server đang sử dụng phiên bản 29.9.0. Cgroup
Driver là systemd, nghĩa là Docker sử dụng systemd làm trình quản lý
nhóm cgroups trên hệ thống này. Tổng cộng có 2 container, trong đó 1
container đang chạy và 1 container đã dừng; không có container ở trạng
thái paused. Máy chủ có 2 Docker images. Đây là các thông số tại thời
điểm chụp ảnh; chúng có thể thay đổi khi tạo, chạy, dừng hoặc xóa
container và image.
Ngoài ra, ảnh cho thấy hệ điều hành máy chủ là Ubuntu 24.04.2 LTS,
kiến trúc x86_64 và có 1 CPU được Docker ghi nhận. Các thông tin bổ
sung này giúp mô tả môi trường chạy Docker, nhưng bốn thông số chính
được yêu cầu là Server Version, Cgroup Driver, số lượng Containers và số
lượng Images đã được liệt kê trong bảng trên.
Kết luận
VM và container đều hỗ trợ đóng gói, triển khai và vận hành ứng dụng
nhưng khác nhau về lớp ảo hóa, chi phí tài nguyên và mức độ cô lập.
Container thường nhẹ và khởi động nhanh nhờ chia sẻ kernel của máy chủ.
Trên Linux, namespaces tạo môi trường nhìn thấy tài nguyên được phân
tách, trong khi cgroups hỗ trợ kiểm soát mức sử dụng CPU và RAM. Đầu ra
docker info cung cấp thông tin quan trọng để kiểm tra phiên bản
Docker, cơ chế quản lý cgroups và trạng thái tổng quan của containers,
images trên máy hiện tại.