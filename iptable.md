# iptables
## Khái niệm 
iptables là ứng dụng firewall miễn phí có sẵn trên các hệ điều hành tùy biến phân phối linux

Chúng được thiết lập các quy tắc riêng để bảo vệ hệ thống, giám sát truy cập

Khi sử dụng máy chủ, iptables sẽ tiến hành thực thi tốt nhất nhiệm vụ ngăn chặn các truy cập không hợp lệ bằng cách sử dụng Netfilter

iptables/Netfilter gồm 2 phần chính là phần Netfilter ở bên trong nhân Linux. Phần còn lại là iptables nằm ở bên ngoài

Netfilter chịu trách nhiệm lọc các gói dữ liệu ở mức IP Netfilter làm việc trực tiếp trong nhân, nhanh và giảm tốc độ của hệ thống. iptables chỉ là Interface cho Netfilter. Cả 2 thành phần này có nhiệm vụ tương tự nhau
## Vai trò và lợi ích của iptables
Tích hợp tốt với kernel của Linux. 

Có khả năng phân tích package hiệu quả. 

Lọc packet dựa vào MAC và một số flag trong TCP Header. 

Cung cấp chi tiết các tùy chọn để ghi nhận sự kiện hệ thống. 

Cung cấp kỹ thuật NAT. 

Có khả năng ngăn chặn một số cơ chế tấn công theo kiểu DoS
## Thành phần và chức năng trong iptables
iptables sẽ giám sát mọi luồng ra vào hệ thống bằng __table__

Mặc định của iptables là cho phép mọi gói tin truyền tự do ra vào

Các __table__ có chứa toàn bộ các quy tắc được thiết lập trước đó, gọi là __chain__

Các rule sẽ lọc, phân loại các data packet ra vào thông qua "netfilter", so sánh từng rule trong khi đi qua tất cả rule có ở iptables

Khi các packet trùng với điều kiện của rule, khi này sẽ được xem là __target__ và áp dụng các lệnh thực thi lên packet này

Trong đó:
- Table: có 4 loại khác nhau
    - __Filter table__: đây là gói quen thuộc là sử dụng nhiều nhất, quyết định xem gói tin có đi tới điểm đích an toàn không
    - __Mangle table__: liên quan đến việc sửa head của gói tin, ví dụ chỉnh sửa giá trị các trường TTL, MTU, Type of Service
    - __NAT table__: cho phép route các gói tin đến các host khác nhau trong mạng NAT table bằng cách thay đổi IP nguồn hoặc IP đích của gói tin. Table này cho phép kết nối đến các dịch vụ không được truy cập trực tiếp được do đang trong mạng NAT
    - __Raw table__: 1 gói tin có thể thuộc một kết nối mới hoặc cũng có thể là của 1 một kết nối đã tồn tại. Table raw cho phép bạn làm việc với gói tin trước khi kernel kiểm tra trạng thái gói tin

![](/images/table_type.jpg)

- Chain: 
    Mỗi table được tạo với một số chains nhất định. Chains cho phép lọc gói tin tại các điểm khác nhau. iptables có thể thiết lập với các chains sau:
    - __INPUT__: rule thuộc chain này áp dụng cho các gói tin ngay trước khi các gói tin được vào hệ thống. Chain này có trong 2 table mangle và filter, Khi có 1 rule nất kì trùng với thì các actionc có thể áp dụng với packet được chọn
    - __PREROUTING__: Các rule thuộc chain này sẽ được áp dụng ngay khi gói tin vừa vào đến Network interface. Chain này chỉ có ở table NAT, raw và mangle
    - __OUTPUT__: Các rule thuộc chain này áp dụng cho các gói tin ngay khi gói tin đi ra từ hệ thống. Chain này có trong 3 table là raw, mangle và filter
    - __FORWARD__: Các rule thuộc chain này áp dụng cho các gói tin chuyển tiếp qua hệ thống. Chain này chỉ có trong 2 table mangle và table
    - __POSTROUTING__: áp dụng cho các gói tin đi network interface. Chain này có trong 2 table mangle và NAT
- Target: hiểu đơn giản là các hành động áp dụng cho các gói tin. Đối với những gói tin trùng khớp điều kiện của rule đươc đưa ra thì sẽ được xếp vào các trường hợp sau:
    - accept: cho phép gói tin đi vào hệ thống
    - drop: loại bỏ gói tin và không gửi phản hồi
    - reject: loại bỏ gói tin đến và phản hồi gói khác cho iptables
    - log: cho phép gói tin và ghi nhận lại trên file log

Đối với những gói tin không khớp với rule nào cả mặc định sẽ được chấp nhận

## Cấu trúc câu lệnh iptables
TARGET-----PROT-----OPT-----IN-----OUT----SOURCE-----DESTINATION

Trong đó:
- _target_ là hành động cần thực thi
- _prot_ là giao thức mà các bên kết nối với nhau 
- _opt:_ tùy chọn thêm bớt điều kiện quét. Thường thì ở phần __OPT__ sẽ để trống __"--"__
- _IN/OUT:_ chỉ định giao thức nào được phép ra vào của packet, như `lo` hay `eth1`, `eth2`,...`any` hoặc `all`
- _source:_ nơi gửi yêu cầu
- _des:_ nơi kết thúc và nhận phản hồi

## Sơ đồ hoạt động của iptables
![](/images/Iptables.gif)

