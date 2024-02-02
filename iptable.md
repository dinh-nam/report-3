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

Lọc packet dựa vào MAC và một số flag trong tcp Header. 

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
    - __Mangle table__: liên quan đến việc sửa header của gói tin, ví dụ chỉnh sửa giá trị các trường TTL, MTU, Type of Service
    - __NAT table__: cho phép route các gói tin đến các host khác nhau trong mạng NAT table bằng cách thay đổi IP nguồn hoặc IP đích của gói tin. Table này cho phép kết nối đến các dịch vụ không được truy cập trực tiếp được do đang trong mạng NAT
    - __Raw table__: 1 gói tin có thể thuộc một kết nối mới hoặc cũng có thể là của 1 một kết nối đã tồn tại. Table raw cho phép bạn làm việc với gói tin trước khi kernel kiểm tra trạng thái gói tin
    - __Security table__: Một số Kernel có thể hỗ trợ thêm cho Security Table, được dùng bởi Selinux từ đó giúp thiết lập các chính sách bảo mật hiệu quả, đánh dấu những gói tin có liên quan đến context của SELinux, nó giúp cho SELinux hoặc các thành phần khác hiểu được context SELinux và xử lý gói tin

![](/images/table_type.jpg)

- Chain: 
    Mỗi table được tạo với một số chains nhất định. Chains cho phép lọc gói tin tại các điểm khác nhau. iptables có thể thiết lập với các chains sau:
    - __INPUT__: rule thuộc chain này áp dụng cho các gói tin ngay trước khi các gói tin được vào hệ thống. Chain này có trong 2 table mangle và filter, Khi có 1 rule nất kì trùng với thì các action có thể áp dụng với packet được chọn
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

## Tổng quan cơ chế xử lí gói tin trong iptables
*Mangle:* chịu trách nhiệm thay đổi các bits chất lượng dịch vụ trong tcp header như TOS (type of service), TTL (time to live), và MARK

*Filter:* chịu trách nhiệm lọc gói dữ liệu. Nó gồm có 3 quy tắc nhỏ (chain) để giúp bạn thiết lập các nguyên tắc lọc gói:

- Forward chain : lọc gói khi đi đến đến các server khác.
- Input chain : lọc gói khi đi vào trong server.
- Output chain: lọc gói khi ra khỏi server

*NAT:* gồm có 2 loại:

- Pre-routing chain: thay đổi __ip-des__ của gói dữ liệu khi cần thiết
- Post-routing chain: thay đổi __ip-source__ của gói dữ liệu khi cần thiết
## Sơ đồ hoạt động của iptables
![](/images/Iptables.gif)

Gói tin đi vào mạng A

Được kiểm tra ở magle-PREROUTING nếu cần

Ở nat-PREROUTING sẽ kiểm tra gói packet có cần DNAT hay ko, nếu có sẽ đổi IP đến và chuyển tiếp

Sau khi qua bước định tuyến sẽ đến với lọc gói:

Nếu gói tin định hướng đi vào mạng được bảo vệ B, khi này nó sẽ được lọc bởi FORWARD chain của filter table

Thay đổi ip-source ở POSTROUTING chain trước khi chuyển tiếp vào mạng bảo vệ B

Ngược lại nếu gói tin đi vào trong firewall, nó sẽ được kiểm tra tại mangle table và INPUT chain

TIếp tục lọc gói tin tại filter table và được chuyển tiếp vào bên trong các chương trình firewall, xử lí dữ liệu cục bộ 

Khi firewall cần gửi dữ liệu ra ngoài. Gói dữ liệu sẽ được qua bước định tuyến và đi qua sự kiểm tra của OUTPUT chain trong mangle table

Tiếp đó là kiểm tra trong OUTPUT chain của nat table để xem DNAT có cần hay không và OUTPUT chain của filter table sẽ kiểm tra gói dữ liệu nhằm phát hiện các gói dữ liệu không được phép gởi đi.

Cuối cùng trước khi gói dữ liệu được đưa ra lại Internet, SNAT and QoS sẽ được kiểm tra trong POSTROUTING chain

## Các tình huống 
** Đường đi của gói tin có destination là ip-server:

Đường đi của packet vẫn như cũ, tại bước lọc packet sẽ là chọn đi vào server

Từ đây, packet tại bước mangle INPUT sẽ được chỉnh sửa xáo trộn trước khi được chuyển tiếp qua bước lọc filter INPUT để đối chiếu với các rule thiết lập từ trước và xem xét đến server hay bị hủy

** Gói tin từ server đi ra :

Từ server , packet được xếp vào bảng raw OUTPUT để kiểm tra trước khi giao vào kernel xử lý. Qua bước mangle OUTPUT packet tiếp tục chỉnh sửa để tránh các rủi ro tiềm ẩn và chuyển cho nat OUTPUT để chỉnh sửa IP nguồn

Sau đó, packet qua bước routing và được xem xét trong trường hợp nat hay mangle sẽ thay đổi packet

Tiếp tục đi qua bảng lọc đầu ra filter OUTPUT để so sánh các rule, qua bước mangle POTSROUTING chỉnh sửa các trường trong header của packet, đối chiếu lần nữa các rule để xem xét gói tin có đủ tiêu chuẩn ACCEPT hay DROP hoặc REJECT,...

Tại bước cuối nat POTSROUTING sẽ thay đổi ip nguồn hoặc ip đích trước khi chuyển tiếp packet ra khỏi bảng, đối với các packet có SNAT hoặc `masquerading` thì nên tránh bước này

** Cách server xử lý gói tin khi có des là IP khác server :

Khi có gói tin đi vào hệ thống mà IP đích là thuộc về các client bên trong thì server sẽ là "proxy" và chuyển hướng đường đi về đúng vị trí cần đến

Trình tự packet đi vào sẽ vẫn như cũ cho đến bước lọc gói tin, sau khi xác định gói cần forward đến nơi khác, sẽ được đẩy vào mangle FROWARD, thay đổi dữ liệu bên trong, lọc đối chiếu các rule trong hệ thống ở filter FORWARD rồi đẩy ra magle POSTROUTING

Xử lý gói tin ở các bước cuối hệ thống như cũ và chuyển ra ngoài để đi vào mạng bảo vệ

![](/images/Iptables-Flow.png)

# tcpdump
## Khái niệm
Công cụ được phát triển nhằm mục đích kiểm tra và nắm bắt các gói mạng đi vào và ra khỏi máy tính. 

Đây là một trong những tiện ích mạng phổ biến nhất để quản lý, khắc phục sự cố mạng và sự cố bảo mật

tcpdum cho phép chặn và hiển thị các gói tin được truyền đi hoặc được nhận trên một mạng mà máy tính có tham gia

Dù có tên tcpdump nhưng nó có thể được sử dụng để kiểm tra lưu lượng không phải tcp bao gồm UDP, ARP,...
## Cài đặt và sử dụng
Thông thường ở OS như Ubuntu công cụ tcpdump sẽ có sẵn

Tuy nhiên vẫn có 1 số version của nó hoặc bản OS khác phải cài đặt thủ công như CentOS hay Kali Linux,...

Ubuntu: `sudo apt-get install tcpdump -y` (dành cho các version cũ)

CentOS: `yum install tcpdump -y`

Để biết được công cụ đã có sẵn hay chưa, chỉ cần kiểm tra version của nó
```
sudo tcpdump --version
```
Sau đó kết quả sẽ trả về thông tin phiên bản nếu công cụ đã được cài sẵn và ngược lại

### ------------------------------------- Bắt gói tin trên đường mạng -------------------------------------------

Khi dùng lệnh tcpdump mà không có bất cứ điều kiện đi kèm, nó sẽ tự bắt toàn bộ các packet đang ra vào chính thiết bị có nối mạng của user 
```
sudo tcpdump
```
Quá trình chỉ dừng khi user hủy trực tiếp bằng tổ hợp Crtl-C, kết quả thống kê sẽ được trả về như sau:

- Packet captured: số lượng gói tin bắt được và xử lý.
- Packet received by filter: số lượng gói tin được nhận bởi bộ lọc.
- Packet dropped by kernel: số lượng packet đã bị dropped bởi cơ chế bắt gói tin của hệ điều hành

Cấu trúc chung của dòng lệnh tcpdump thường là:
```
time-stamp src > dst: flags data-seqno ack window urgent options
```
time-stamp: thời gian bắt packet

src / dst: ip nguồn và ip đích

flags: bao gồm

    S(SYN): dùng trong quá trình bắt tay của giao thức tcp
    ACK: gói phản hồi từ ip đích thông báo nhận thành công
    F(FIN): đóng kết nối tcp
    P(PUSH): đặt ở cuối để đánh dấu việc truyền dữ liệu
    R(RST): dùng khi thiết lập lại đường truyền
data-seqno: số sequence của packet hiện tại

ack: Mô tả số sequence mong muốn tiếp theo của gói tin do bên gửi truyền

window: Vùng nhớ đệm có sẵn theo hướng khác trên kết nối này

urgent: có gói dữ liệu khẩn bên trong

option : tùy chọn đi kèm

    -i : chỉ định cổng giao tiếp mạng (eth, lo,...)
    -c : chỉ định số packet cần giám sát
    -A : chỉ định hiển thị packet ở form ASCII
    -D : hiển thị toàn bộ cổng giao tiếp đang hoạt động
    -xx : bắt dữ liệu từng packet, bao gồm mức liên kết header ở cả form hex và ASCII
    -w : tính năng cho phép bắt packet và lưu file dữ liệu có đuôi ".pcap"
    -r : để có thể đọc và phân tích file có đuôi ".pcap"
    -n : bắt gói tin trên cổng giao tiếp được chỉ định,
        option "port" để chỉ định thêm cổng kết nối
        option "tcp" chỉ bắt các packet dựa trên giao thức này
Ngoài ra có thể kết hợp cùng các bộ lọc khác qua 3 cách:
- && : thực thi nhiều lệnh
- ||: chỉ một trong số các lệnh
- ! : không thực thi lệnh chỉ định

1 số ví dụ sử dụng lệnh của tcpdump:

Bắt gói tin trên cổng eth0 và số gói chỉ dịnh là 6:
```
sudo tcpdump -i eth0 -c 6
```
Bắt gói tin tại host chỉ định có IP là 172.19.11.101 và số packet là 5:
```
sudo tcpdump -n host 172.19.11.101 -c 5
```
Bắt gói tin trên port 80:
```
sudo tcpdump -n port 80
```
Bắt gói tin truyền từ ip-nguồn/ip-đích:
```
sudo tcpdump -n src/dst 172.19.11.101
```
Bắt gói tin trên giao thức được chỉ định là udp:
```
sudo tcpdump -n udp
```
## TRACE module
Cũng như wiresakrk hay tcpdump, TRACE có thể giám sát, đọc và phân tích các gói ra vào host nhưng ở cả chain và table trong netfilter

Khuyến khích sử dụng ổn định nhất trong CentOS

Đầu tiên cần tải module xem log file ipv4 netfilter :
```
modprobe nf_log_ipv4
```
Bật ghi nhật ký cho IPv4 (AF Family 2) :
```
sysctl net.netfilter.nf_log.2=nf_log_ipv4
```
Ở bước này khi user thực hiện thao tác và các packet bắt đầu được xử lý và ra vào thông qua iptables thì file log sẽ ghi lại quá trình từng bước trong hệ thống iptables

Ngoài ra sau bước này có option xem thông tin từ log của kernel:
```
cat /etc/rsyslog.conf | grep -e "^kern"
systemctl restart rsyslog
```
Và chạy lại `rsyslog`

Xem các rule đã được apply vào bảng:
```
iptables -t raw -L
```
Lúc này đã có thể đặt rule TRACE trong iptables để theo dõi các packet

Ví dụ: để xem các gói đi từ host 192.168.0.121 đến host có IP là 192.168.0.144
```
iptables -t raw -A OUTPUT -p tcp -j TRACE
iptables -t raw -A PREROUTING -p tcp -j TRACE
```
Từ đây khi kết nối và thực hiện tcp giữa các host, user sẽ thấy được quá trình  liên lạc giữ các host diễn ra

Đầu tiên từ host 192.168.0.121 thực hiện ping imcp qua 192.168.0.144

![](/images/ping%20host.png)

Tại host 192.168.0.144 mở file log TRACE để xem lại đường đi 

![](/images/file%20log%20tcp.png)

```
$ sudo iptables -t raw -A PREROUTING -p tcp --destination 192.168.0.144 -j TRACE
$ sudo iptables -t raw -A PREROUTING -p tcp --destination 192.168.0.144 -j TRACE
```
2 lệnh này cho phép TRACE các gói khi đi qua firewall vào bên trong hệ thống
từ các host bất kỳ bên ngoài

Ví dụ: từ host 192.168.0.143 bắt gói tin tcp kết nối 1.1.1.1

![](/images/ping%201.1.1.1.png)

Trong kết quả phân tích này:

Ở ip 192.168.0.143 gửi icmp đến host 1.1.1.1 và từ host sẽ gửi phản hồi về ip source, tại ip source dùng TRACE bắt gói tin thông qua iptables

Cấu trúc của kết quả TRACE khi xem ở /log/message sẽ hiển thị các thông tin đầy đủ về tiến trình qua từng bước trong flow

Xử lý gói tin ở mục nào, cổng giao tiếp là gì, MAC của thiết bị đang dùng TRACE, ip source và ip des, độ dài gói tin, type of service và độ ưu tiên, Time to live, ID tiến trình, giao thức thực hiện và các thông tin liên quan gói tin

Vì thực hiện TRACE ở ip 192.168.0.143 nên gói tin có source là 1.1.1.1 gửi phản hồi về host hiện tại

Trình tự gói tin khi đi vào host và qua iptables thì có thể thấy đang đi vào mangle PREROUTING sửa đổi các header trong gói tin

Sau đó chuyển sang mangle INPUT: sửa đổi header khi chuẩn bị đi vào mạng bảo vệ

Chuyển tiếp qua bước lọc security INPUT để đảm bảo gói tin này là an toàn

** Tương tự TRACE cũng sẽ bắt gói tin đi từ host bên trong ra ngoài Internet và nhận phản hồi từ bên ngoài
```
iptables -t raw -A PREROUTING -s 192.168.0.121 -d 192.168.0.143 -j TRACE.
iptables -t raw -A OUTPUT -s 192.168.0.121 -d 192.168.0.143 -j TRACE.
```
Có thể đặt rule tương tự hoặc chỉ cần rule bắt gói tin tcp như ví dụ trước

## Thiết lập rule iptables

![](/images/iptables-rule.png)

![](/images/insert-rule.png)

+ Cho phép/Chặn IPX truy cập đến IP dest A.B.C.D port YYY
```
iptables -A INPUT -p tcp -s 192.168.0.121 -d 192.168.0.143 --dport 80 -j DROP
iptables -A INPUT -p tcp -s 192.168.0.121 -d 192.168.0.143 --dport 443 -j ACCEPT
```
+ Cho phép/Chặn tất cả ip mới truy cập đến IP dest A.B.C.D port YYY
```
iptables -A INPUT -p tcp -d 192.168.0.143 --dport 80 -j DROP
iptables -A INPUT -p tcp -d 192.168.0.143 --dport 443 -j ACCEPT
```
+ Cho phép/Chặn ip Y.J.K.F truy cập đến IP dest A.B.C.D port YYY với TTL 128,64 và Length 1000
```
iptables -A INPUT p tcp -s 192.168.0.121 -d 192.168.0.143 --dport 80 -m length --length 1000 --match ttl --ttl-eq 64 -j REJECT 

iptables -A INPUT -p tcp -s 192.168.0.121 -d 192.168.0.143 --dport 443 -m length --length 1000 --match ttl --ttl-eq 128 -j ACCEPT
```
Cho phép/Chặn IPX truy cập từ IP src A.B.C.D port YYY
```
iptables -A OUTPUT -p tcp -s 192.168.0.121 --dport 80 -j DROP
iptables -A OUTPUT -p tcp -s 192.168.0.121 --dport 443 -j ACCEPT
```
__Lưu ý__: ưu tiên các rule DROP trước các rule ACCEPT để lọc đầy đủ các điều kiện ban đầu hơn
### Đặt comment cho 1 iptables rules bất kỳ.
 
1. Cách thêm phần chú thích vào rule thường sẽ có cấu trúc sau:
```
iptables -m comment --comment "My comments here"
```
Ví dụ:

`iptables -A INPUT -i eth1 -m comment --comment "my LAN - " -j DROP`

_Một lần thêm comment tối đa được 256 kí tự_

_Vị trí đặt comment chủ yếu nẳm trước phần [target]_

_Khi show bảng table sẽ có cột chú thích được thêm đi kèm_

Sau khi thêm rule cũng như chèn comment thì thực hiện xác nhận lại:
```
iptables -t filter -L INPUT -n
```
2. Chèn chú thích vào rule đang có sẵn
```
iptables -R chain [rule-number] rule-specification [comment]
```
Trong đó -R: lệnh thay thế

rule-number: số thứ tự của rule trong bảng iptables

Ví dụ: `iptables -A INPUT -p tcp --dport 25 -j DROP` - chặn mọi kết nối ở port 25
Để thêm comment sẽ có syntax như sau:
```
iptables -R INPUT 11 -p tcp --dport 25 -j DROP -m comment --comment "Block port 25"
iptables -t filter -L INPUT -n --line-number
```
## Kết thúc