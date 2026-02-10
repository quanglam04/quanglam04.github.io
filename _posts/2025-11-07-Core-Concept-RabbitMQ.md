---
title: "Các khái niệm cốt lõi cần nắm khi làm việc với RabbitMQ"
date: 2025-11-07 01:17:00  +0700
categories: [message broker]
tags: [rabbitmq]
---


_Nguồn: Medium_

**Giới thiệu**

Trong hầu hết các ứng dụng hiện đại, chúng ta đang chuyển sang kiến trúc **microservices**. Vì kiến trúc microservices bao gồm nhiều dịch vụ độc lập, nên các kết nối giữa các dịch vụ này trở nên **cực kỳ lớn**. Do đó, hệ thống trở nên **phức tạp**.

Chúng ta không thể dựa vào các hệ thống dựa trên yêu cầu/phản hồi (request/response)trong tình huống này vì các ứng dụng thường phải đợi phản hồi từ dịch vụ khác. Một máy khách kích hoạt một dịch vụ vốn tốn thời gian, chẳng hạn như một dịch vụ đòi hỏi tài nguyên tính toán đáng kể, có thể làm tình hình trầm trọng hơn. Điều này có thể ảnh hưởng đến trải nghiệm người dùng nếu máy khách là một ứng dụng giao diện người dùng (front-end) vì giao diện người dùng trở nên không phản hồi.

Tất cả những yếu tố này tạo ra nhu cầu giao tiếp bất đồng bộ (asynchronous communication) giữa các microservices, và Hệ thống Dựa trên Tin nhắn (Message Based Systems) có thể trợ giúp. Trong một hệ thống như vậy, một thành phần mới gọi là bộ phận trung gian (intermediary) được giới thiệu để tạo điều kiện giao tiếp giữa hai ứng dụng/dịch vụ thay vì thiết lập giao tiếp trực tiếp giữa chúng. Thành phần ở giữa này được gọi là **Message Broker**.

<p align="center">
  <img src="/assets/images/rabbit-mq/1.png" alt="Image title_1" />
</p>

Message Broker giúp khử khớp nối (decoupling) các ứng dụng vì chúng không tương tác trực tiếp với nhau. Mỗi ứng dụng chỉ cần tuân theo định dạng dữ liệu của Message Broker này và không cần xử lý các sắc thái khi tương tác với các dịch vụ khác nhau. Do đó, nó đơn giản hóa việc kết hợp các ứng dụng không đồng nhất (heterogeneous applications) vào hệ thống.

Message Broker cung cấp khả năng **xếp hàng** (queue) các tin nhắn, cho phép các máy chủ web phản hồi yêu cầu một cách nhanh chóng thay vì bị buộc phải thực hiện các quy trình **tốn nhiều tài nguyên** ngay lập tức có thể làm chậm thời gian phản hồi. Xếp hàng tin nhắn cũng tốt khi bạn muốn **phân phối** một tin nhắn đến nhiều người tiêu thụ (consumers) hoặc để **cân bằng tải** giữa các workers.

## RabbitMQ

RabbitMQ là một Message Broker mã nguồn mở cực kỳ phổ biến được sử dụng để xây dựng các hệ thống dựa trên tin nhắn. Mặc dù RabbitMQ hỗ trợ nhiều giao thức, giao thức được sử dụng phổ biến nhất là AMQP.

**AMQP 0-9-1 (Advanced Message Queuing Protocol)** là một giao thức nhắn tin cho phép các ứng dụng máy khách tuân thủ giao tiếp với các bộ môi giới phần mềm trung gian nhắn tin tuân thủ. Đây là một giao thức lớp ứng dụng truyền dữ liệu dưới dạng nhị phân. Trong ứng dụng này, dữ liệu được gửi dưới dạng khung (frames).

<p align="center">
  <img src="/assets/images/rabbit-mq/2.png" alt="Image title_1" />
</p>


## Key Concept

- **Producer (Nhà sản xuất):** Ứng dụng gửi tin nhắn.
- **Consumer (Người tiêu thụ):** Ứng dụng nhận tin nhắn.
- **Queue (Hàng đợi):** Lưu trữ các tin nhắn được tiêu thụ bởi các ứng dụng.
- **Connection (Kết nối):** Kết nối TCP giữa ứng dụng của bạn và broker RabbitMQ.
- **Channel (Kênh):** Các kết nối nhẹ (lightweight connections) chia sẻ một kết nối TCP duy nhất. Việc xuất bản (Publishing) hoặc tiêu thụ (consuming) tin nhắn từ một hàng đợi được thực hiện qua một kênh.
- **Exchange (Bộ trao đổi):** Nhận tin nhắn từ các producers và đẩy chúng đến các queues tùy thuộc vào các quy tắc được xác định bởi loại exchange. Một queue phải được liên kết (bound) với ít nhất một exchange để nhận tin nhắn.
- **Binding (Liên kết):** Bindings là các quy tắc mà exchanges sử dụng (cùng với những thứ khác) để định tuyến tin nhắn đến các queues
- **Routing key (Khóa định tuyến):** Một khóa mà exchange sử dụng để quyết định cách định tuyến tin nhắn đến các queues. Hãy coi routing key như là một địa chỉ cho tin nhắn.
- **Users (Người dùng):** Có thể kết nối với RabbitMQ bằng tên người dùng và mật khẩu đã cho. Người dùng có thể được gán các quyền như quyền đọc, viết và cấu hình đặc quyền trong instance. Người dùng cũng có thể được gán quyền cho các virtual host cụ thể.
- **Vhost, virtual host (Host ảo):** Virtual host cung cấp khả năng nhóm logic và tách biệt tài nguyên. Người dùng có thể có các quyền khác nhau đối với các vhost khác nhau, và queues và exchanges có thể được tạo để chúng chỉ tồn tại trong một vhost.

## Luồng Tin nhắn trong RabbitMQ

<p align="center">
  <img src="/assets/images/rabbit-mq/3.png" alt="Image title_1" />
</p>

1. Producer xuất bản tin nhắn tới exchange thông qua một channel được thiết lập giữa chúng tại thời điểm ứng dụng khởi động.
2. Exchange nhận tin nhắn và tìm các bindings thích hợp dựa trên các thuộc tính tin nhắn và loại exchange.
3. Binding được chọn sau đó được sử dụng để định tuyến tin nhắn đến các queues đã định.
4. Tin nhắn nằm trong queue cho đến khi được consumer xử lý.
5. Consumers nhận tin nhắn bằng cách sử dụng các channels thường được thiết lập khi ứng dụng khởi động.

## Các Loại Exchange

Exchanges là các thực thể nhận tin nhắn. Exchanges lấy một tin nhắn và định tuyến nó vào không hoặc nhiều queues. Thuật toán định tuyến được sử dụng phụ thuộc vào loại exchange và các quy tắc được gọi là bindings.

<p align="center">
  <img src="/assets/images/rabbit-mq/4.png" alt="Image title_1" />
</p>

Có bốn loại exchanges:

- **Direct (Mặc định):** Tin nhắn được định tuyến đến các queues có binding key khớp với routing key của tin nhắn. Nó chủ yếu được sử dụng cho việc định tuyến unicast (unicast routing) tin nhắn.
- **Fanout:** Nó định tuyến tin nhắn đến tất cả các queues được liên kết với nó, và routing key bị bỏ qua.
- **Topic:** Topic exchange thực hiện một khớp ký tự đại diện (wildcard match) giữa routing key và routing pattern được chỉ định trong binding.
- **Header:** Header exchange sử dụng các thuộc tính tiêu đề tin nhắn (message header attributes) để định tuyến.

## Consumer Acknowledgements và Publisher Confirms

Vì tin nhắn được gửi qua mạng không được đảm bảo đến đích, RabbitMQ cung cấp một cơ chế xác nhận việc gửi và xử lý.
- Xác nhận xử lý việc gửi từ consumer đến broker được gọi là consumer acknowledgment (xác nhận của người tiêu thụ)
- Xác nhận của broker cho publisher rằng tin nhắn đã được nhận được gọi là publisher confirms (xác nhận của nhà xuất bản).

Cả hai đều có thể là tích cực (positive) và tiêu cực (negative)
- Xác nhận tích cực từ consumers có nghĩa là một tin nhắn đã được xử lý thành công, và tiêu cực có nghĩa là thất bại trong việc xử lý.
- Điều tương tự cũng áp dụng trong trường hợp publisher confirms.

RabbitMQ hỗ trợ việc xác nhận nhiều tin nhắn cùng một lúc.

## Prefetching Messages

Vì việc gửi và xác nhận tin nhắn đều được thực hiện bất đồng bộ, có thể có nhiều hơn một tin nhắn "đang bay" (in flight) trên một channel tại bất kỳ thời điểm nào. Điều này có nghĩa là chúng ta có một cửa sổ trượt (sliding window) các lần gửi chưa được xác nhận.

Đối với hầu hết các consumers, nên giới hạn kích thước của cửa sổ này để tránh vấn đề tăng trưởng bộ đệm không giới hạn (unbounded buffer/heap growth) ở phía consumer. Điều này được thực hiện bằng cách đặt giá trị "prefetch count".

- Giá trị này xác định số lượng tối đa các lần gửi chưa được xác nhận được phép trên một channel.

- Khi số lượng đạt đến số lượng đã cấu hình, RabbitMQ sẽ ngừng gửi thêm tin nhắn trên channel cho đến khi ít nhất một trong số các tin nhắn đang chờ được xác nhận.

- Điều này có thể được sử dụng như một kỹ thuật cân bằng tải đơn giản hoặc để cải thiện thông lượng nếu tin nhắn có xu hướng được xuất bản theo lô.

- Nói chung, tăng prefetch sẽ cải thiện tốc độ gửi tin nhắn đến consumers.

## Best Practices

- Đảm bảo kích thước queue của bạn luôn nhỏ. Queue lớn sẽ tạo ra tải nặng lên việc sử dụng RAM. Để giải phóng RAM, RabbitMQ đẩy tin nhắn xuống đĩa, do đó ảnh hưởng xấu đến tốc độ xếp hàng và hiệu suất của broker.

- Giới hạn kích thước queue bằng cách đặt TTL (Time-to-Live) cho tin nhắn hoặc độ dài tối đa của queue. Điều này nên được sử dụng trong các trường hợp mà consumers có thể chịu được mất mát tin nhắn.

- Đảm bảo thực hiện xử lý tin nhắn ở phía consumer một cách bất đồng bộ nếu có thể và xác nhận lại ngay lập tức. Điều này ảnh hưởng đến độ sâu queue

- Tái sử dụng các kết nối TCP và sử dụng channels để ghép kênh (multiplex) kết nối giữa các luồng (threads). Lý tưởng nhất là mỗi tiến trình nên mở một kết nối duy nhất và sử dụng nhiều channels trong kết nối đó cho các luồng khác nhau trong ứng dụng của bạn.

- Giới hạn số lượng kết nối. Nhiều kết nối sẽ tạo ra nhiều số liệu (metrics) làm tăng tiêu thụ CPU. Nếu không thể giảm số lượng kết nối, hãy tăng khoảng thời gian thu thập thống kê.

- Không mở và đóng kết nối thường xuyên. Điều này có thể dẫn đến độ trễ cao hơn vì nhiều gói TCP sẽ được gửi và nhận. Các kết nối nên có tuổi thọ dài (long-lived).

- Nếu ứng dụng của bạn không thể chịu được mất mát tin nhắn, queue của bạn nên được khai báo là bền vững (durable), và tin nhắn nên được gửi với chế độ gửi persistent (persistent delivery mode).

- Nếu producer và consumer nằm trong một ứng dụng duy nhất, chúng ta nên sử dụng các kết nối riêng biệt cho cả hai để giữ chúng cô lập khỏi luồng điều khiển của nhau.
