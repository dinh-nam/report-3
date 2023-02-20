# IP-table
## Khái niệm 
IPtable là ứng dụng firewall miễn phí có sẵn trên các hệ điều hành tùy biến phân phối linux

Chúng được thiết lập các quy tắc riêng để bảo vệ hệ thống, giám sát truy cập

Khi sử dụng máy chủ, Iptables sẽ tiến hành thực thi tốt nhất nhiệm vụ ngăn chặn các truy cập không hợp lệ bằng cách sử dụng Netfilter

Iptables/Netfilter gồm 2 phần chính là phần Netfilter ở bên trong nhân Linux. Phần còn lại là Iptables nằm ở bên ngoài

Netfilter chịu trách nhiệm lọc các gói dữ liệu ở mức IP Netfilter làm việc trực tiếp trong nhân, nhanh và giảm tốc độ của hệ thống. Iptables chỉ là Interface cho Netfilter. Cả 2 thành phần này có nhiệm vụ tương tự nhau
## Vai trò và lợi ích của IPtable
Tích hợp tốt với kernel của Linux. 

Có khả năng phân tích package hiệu quả. 

Lọc package dựa vào MAC và một số cờ hiệu trong TCP Header. 

Cung cấp chi tiết các tùy chọn để ghi nhận sự kiện hệ thống. 

Cung cấp kỹ thuật NAT. 

Có khả năng ngăn chặn một số cơ chế tấn công theo kiểu DoS
## Thành phần và chức năng trong IPtable
IPtable sẽ giám sát mọi luồng ra vào hệ thống bằng __table__

Các __table__ có chứa toàn bộ các quy tắc được thiết lập trước đó, gọi là __chain__

các rule sẽ lọc, phân loại các data packet ra vào thông qua netfilter

Khi các packet trùng với điều kiện của rule, khi này sẽ được xem là __target__

