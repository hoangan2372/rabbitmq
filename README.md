# 1. Giới thiệu RabbitMQ
## a. RabbitMQ là gì?.
- RabbitMQ là một phần mềm mã nguồn mở cung cấp giải pháp message broker, được sử dụng để xử lý, lưu trữ và phân phối các thông điệp (messages) giữa các ứng dụng và hệ thống khác nhau. RabbitMQ hỗ trợ nhiều giao thức như Advanced Message Queuing Protocol (AMQP), Simple (or Streaming) Text Oriented Message Protocol (STOMP), Message Query Telemetry Transport (MQTT), HTTP và WebSockets, giúp cho việc giao tiếp giữa các ứng dụng và hệ thống trở nên dễ dàng và linh hoạt hơn.
- RabbitMQ cũng cung cấp tính năng Work Queue (Task Queue), giúp cho các ứng dụng có thể xử lý các công việc phức tạp một cách hiệu quả hơn. Các công việc được gửi đến RabbitMQ và lưu trữ trong các hàng đợi (queues), sau đó các worker có thể lấy các công việc từ các hàng đợi này để xử lý. RabbitMQ hỗ trợ nhiều tính năng, giúp cho việc sử dụng Work Queue trở nên dễ dàng và tiện lợi hơn
## b. Architecture cơ bản của message queue
![amq1.png](/assets/booking/rabbitmq/amq1.png)
- Producer : là ứng dụng client, tạo message và publish tới broker.
- Consumer : là ứng dụng client khác, kết nối đến queue, Subscribe (đăng ký) và xử lý (consume) message.
- Broker (RabbitMQ) : nhận message từ Producer, lưu trữ chúng an toàn trước khi được lấy từ Consumer.

## c. Vấn đề và tại sao sử dụng RabbitMQ:
- Giả sử chúng ta có một web application cho phép user đăng ký thông tin. Sau khi đăng ký, hệ thống sẽ xử lý thông tin, generate PDF và gửi email lại user. Những công việc này hệ thống cần nhiều thời gian để xử lý, chẳng hạn. Nếu xử lý theo cách thông thường thì user sẽ phải chờ đến khi hệ thống xử lý hoàn thành, và nếu hệ thống có hàng nghìn user truy cập cùng lúc sẽ gây quá tải server.
- Đối với những loại công việc thế này, chúng ta sẽ xử lý nó bằng cách sử dụng message queue: Sau khi user nhập đầy đủ các thông tin trên web interface, web application sẽ tạo một Message “Generate PDF” chứa đầy đủ thông tin cần thiết và gửi nó vào Queue của RabbitMQ.
- Và một số vấn đề khác:
-- Đối với các hệ thống sử dụng kiến trúc microservice thì việc gọi chéo giữa các service khá nhiều, khiến cho luồng xử lý khá phức tạp.
-- Mức độ trao đổi data giữa các thành phần tăng lên khiến cho việc lập trình trở nên khó khăn hơn (vấn đề maintain khá đau đầu).
-- Khi phát triển ứng dụng làm sao để các lập trình viên tập trung vào các domain, business logic thay vì các công việc trao đổi ở tầng infrastructure.
-- Với các hệ thống phân tán, khi việc giao tiếp giữa các thành phần với nhau đòi hỏi chúng cần phải biết nhau. Nhưng điều này gây rắc rối cho việc viết code. Một thành phần phải biết quá nhiều dẫn đến rất khó maintain, debug.

#### Flow đầy đủ sử dụng RabbitMQ để giải quyết vấn đề trên
![amq2.png](/assets/booking/rabbitmq/amq2.png)
- (1) User gửi yêu cầu tạo PDF đến web application.
- (2) Web application (Producer) gửi tin nhắn đến RabbitMQ bao gồm dữ liệu từ request như tên và email.
- (3) Một Exchange chấp nhận các tin nhắn từ Producer và định tuyến chúng đến Queue (hàng đợi) để tạo PDF.
- (4) Ứng dụng xử lý PDF (Consumer) nhận Message từ Queue và bắt đầu xử lý PDF.

## d. Một số Lợi ích và tính năng chính của RabbitMQ
- Transparency: Một producer không cần phải biết consumer. Nó chỉ việc gửi message đến các queue trong message broker. Consumer chỉ việc đăng ký nhận message từ các queue này.
- Many Client: Vì producer giao tiếp với consumer trung gian qua message broker nên dù producer và consumer có khác biệt nhau về ngôn ngữ thì giao tiếp vẫn thành công. Hiện nay rabbitmq đã hỗ trợ rất nhiều ngôn ngữ khác nhau.
- Asynchronous (bất đồng bộ): Producer không thể biết khi nào message đến được consumer hay khi nào message được consumer xử lý xong. Đối với producer, đẩy message đến message broker là xong việc. Consumer sẽ lấy message về khi nó muốn. Đặc tính này có thể được tận dụng để xây dựng các hệ thống lưu trữ và xử lý log, background task.
- Flexible Routing: message được định tuyến (route) thông qua Exchange trước khi đến Queue. RabbitMQ cung cấp một số loại Exchange thường dùng, chúng ta cũng có thể định nghĩa riêng Exchange cho riêng mình.
- Management & Monitoring: cung cấp HTTP API, command-line tool và UI để quản lý và giám sát.
...

# 2. Những khái niệm cơ bản trong RabbitMQ:
- Producer: Ứng dụng gửi message.
- Consumer: Ứng dụng nhận message.
- Exchange: Là nơi nhận message được publish từ Producer và đẩy chúng vào queue dựa vào quy tắc của từng loại Exchange. Để nhận được message, queue phải được nằm (binding) trong ít nhất 1 Exchange.
- Queue: Lưu trữ messages.
- Message: Thông tin truyền từ Producer đến Consumer qua RabbitMQ.
- Connection: Một kết nối TCP giữa ứng dụng và RabbitMQ broker.
- Channel: Một kết nối ảo trong một Connection. Việc publishing hoặc consuming message từ một queue đều được thực hiện trên channel.
- Binding: là quy tắc (rule) mà Exchange sử dụng để định tuyến Message đến Queue. Đảm nhận nhiệm vụ liên kết giữa Exchange và Queue.
- Routing key: Một key mà Exchange dựa vào đó để quyết định cách để định tuyến message đến queue. Có thể hiểu nôm na, Routing key là địa chỉ dành cho message.
- AMQP (Advance Message Queuing Protocol): là giao thức truyền message được sử dụng trong RabbitMQ.
...

# 3. RabbitMQ hoạt động như thế nào?
![amq3.png](/assets/booking/rabbitmq/amq3.png)
- (1) Producer đẩy message vào Exchange. Khi tạo Exchange, bạn phải mô tả nó thuộc loại gì. Các loại Exchange sẽ được giải thích phía dưới.
- (2) Sau khi Exchange nhận Message, nó chịu trách nhiệm định tuyến message. Exchange sẽ chịu trách nhiệm về các thuộc tính của Message, ví dụ routing key, loại Exchange.
- (3) Việc binding phải được tạo từ Exchange đến Queue (hàng đợi). Trong trường hợp này, ta sẽ có hai binding đến hai hàng đợi khác nhau từ một Exchange. Exchange sẽ định tuyến Message vào các hàng đợi dựa trên thuộc tính của của từng Message.
- (4) Các Message nằm ở hàng đợi đến khi chúng được xử lý bởi một Consumer.
- (5) Consumer xử lý Message nhận từ Queue.

# 4. Exchange và các loại Exchange:
- Message không được publish trực tiếp vào Queue; thay vào đó, Producer gửi message đến Exchange.
- Exchange là nơi mà các message được gởi. Exchange nhận tin nhắn và định tuyến nó đến 0 hoặc nhiều Queue với sự trợ giúp của các ràng buộc (binding) và các khóa định tuyến (routing key).
- Thuật toán định tuyến được sử dụng phụ thuộc vào Loại Exchange và quy tắc (còn gọi là ràng buộc hay binding).
![amq4.png](/assets/booking/rabbitmq/amq4.png)
## Các loại Exchange
 > Có 4 loại Exchange: Direct, Fanout, Topic, Headers.
- Lựa chọn các exchange type khác nhau sẽ dẫn đến các đối xử khác nhau của message broker với tin nhắn nhận được từ producer.
- Exchange được bind (liên kết) đến một số Queue nhất định.

| EXCHANGE TYPE | TÊN EXCHANGE MẶC ĐỊNH |
|-------|-------|
| Direct exchange | (Empty string) hoặc amq.direct |
| Fanout exchange | amq.fanout|
| Topic exchange | amq.topic |
| Headers exchange | amq.match (và amq.headers trong RabbitMQ)|

- Ngoài Exchange type, Exchange còn định nghĩa một số thuộc tính:
-- Name: tên Exchange.
-- Durability: thời gian tồn tại khi broker restart
-- Auto-delete: exchange bị xóa khi hàng đợi cuối cùng không còn binding từ nó.
-- Arguments: các tham số không bắt buộc, được sử dụng bởi các plugin và các tính năng dành riêng cho broker.

### (1) Direct Exchange: 
- Định tuyến message đến Queue dựa vào routing key
- Thường được sử dụng cho việc định tuyến tin nhắn unicast-đơn hướng (mặc dù nó có thể sử dụng cho định tuyến multicast-đa hướng).
- Một Exchange không xác định tên (empty string), đây là loại Default Exchange, một dạng đặc biệt của là Direct Exchange. Default Exchange được liên kết ngầm định với mọi Queue với routing key bằng với tên Queue.
- Direct Exchange hữu ích khi muốn phân biệt các thông báo được publish cho cùng một exchange bằng cách sử dụng một mã định danh chuỗi đơn giản.
![amq5.png](/assets/booking/rabbitmq/amq5.png)
- Ví dụ, nếu hàng đợi (Queue) gắn với một exchange có binding key là "pdf_create", message được đẩy vào exchange với routing key là "pdf_create" sẽ được đưa vào hàng đợi này.

### (2) Fanout Exchange:
- Fanout exchange định tuyến message (copy message) tới tất cả queue mà nó được bind, với bất kể một routing key nào. 
- Giả sử, nếu nó N queue được bind bởi một Fanout exchange, khi một message mới published, exchange sẽ định tuyến message đó tới tất cả N queues.
![amq6.png](/assets/booking/rabbitmq/amq6.png)
- Exchange Fanout hữu ích với trường hợp ta cần một dữ liệu được gửi tới nhiều ứng dụng khác nhau với cùng một message nhưng cách xử lý ở ứng dụng là khác nhau.

### (3) Topic Exchange (Publish/Subscribe):
- Topic exchange định tuyến message tới một hoặc nhiều queue dựa trên sự trùng khớp giữa routing key và pattern.
- Topic exchange thường sử dụng để thực hiện định tuyến thông điệp multicast. Ví dụ một vài trường hợp sử dụng:
-- Cập nhật tin tức liên quan đến một category hoặc gắn tag.
-- Phân phối dữ liệu liên quan đến vị trí địa lý cụ thể.
...
![amq7.png](/assets/booking/rabbitmq/amq7.png)
- Một topic exchange sẽ sử dụng "**wildcard**" để gắn routing key với một routing pattern được khai báo trong binding. Consumer có thể đăng ký những topic mà nó quan tâm.
- Ý nghĩa các "**wildcard**" được sử dụng là:
-- "*": có nghĩa là **chính xác một từ** được phép.
-- "#": có nghĩa là **0 hoặc nhiều từ** được phép.
-- ".": có nghĩa là dấu phân cách từ. Nhiều từ chính được phân tách bằng dấu phân cách dấu chấm.
- Ví dụ: 
**topic.*** : được đăng ký bởi tất cả những key với pattern bắt đầu bằng "**topic**" và theo sau là chính xác một từ bất kỳ. 
&nbsp;&nbsp;&nbsp;Ví dụ hợp lệ: topic.task1, topic.task2
&nbsp;&nbsp;&nbsp;Ví dụ không hợp lệ: topic.task1.abc , topic.task2.abc.xyz
**topic.*.final**: được đăng ký bởi tất cả những key với pattern bắt đầu bằng "**topic**", theo sau là chính xác một từ bất kỳ và kết thúc là "**final**".
&nbsp;&nbsp;&nbsp;Ví dụ hợp lệ: topic.task1.final, topic.task2.final
&nbsp;&nbsp;&nbsp;Ví dụ không hợp lệ: topic.task1.abc.final , topic.task2
**topic.#** : được đăng ký bởi tất cả các key bắt đầu với "**topic**"
&nbsp;&nbsp;&nbsp;Ví dụ hợp lệ: topic, topic.task1, topic.task2.abc
&nbsp;&nbsp;&nbsp;Ví dụ không hợp lệ: task1.topic, abc.topic.task2
**topic.#.final**: được đăng ký bởi tất cả những key với pattern bắt đầu bằng "**topic**", theo sau là từ bất kỳ và kết thúc là "**final**".
&nbsp;&nbsp;&nbsp;Ví dụ hợp lệ: topic.final, topic.task2.final, topic.task1.task2.final
&nbsp;&nbsp;&nbsp;Ví dụ không hợp lệ: topic.task1, topic.final.abc
![amq8.png](/assets/booking/rabbitmq/amq8.png)
 ### (4) Headers Exchange:
- Header exchange được thiết kế để định tuyến với nhiều thuộc tính, để dàng thực hiện dưới dạng header của message hơn là routing key. Header exchange bỏ đi routing key mà thay vào đó định tuyến dựa trên header của message.
![amq9.png](/assets/booking/rabbitmq/amq9.png)

### (5) Dead Letter Exchange:
- Dead Letter Exchange là một Exchange bình thường, có thể là một trong 4 loại Exchange
- Nếu không tìm thấy hàng đợi phù hợp cho tin nhắn, tin nhắn sẽ tự động bị hủy. RabbitMQ cung cấp một tiện ích mở rộng AMQP được gọi là “Dead Letter Exchange”. Cung cấp chức năng để chụp các tin nhắn không thể gửi được.

# 5. Implementing RabbitMQ in ASP.NET Core sử dụng thư viện RabbitMQ.Client.Core.DependencyInjection
## a. RabbitMQ.Client.Core.DependencyInjection là gì?
- RabbitMQ.Client.Core.DependencyInjection là một gói phần mềm giúp triển khai các ứng dụng RabbitMQ dễ dàng hơn bằng cách tích hợp DI.
- Hướng dẫn sử dụng [tại đây](https://github.com/AntonyVorontsov/RabbitMQ.Client.Core.DependencyInjection/blob/feature/documentation/docs/readme.md)
## b. Setting up RabbitMQ trên docker:
- Run RabbitMQ server in Docker by cmd: `docker run -d --hostname my-rabbitmq-server --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.11.13-management`

Chúng ta sử dụng rabbitmq:3.11.13-management từ DockerHub, cái mà sẽ cung cấp UI Dashboard, available trên port 15672 (mở localhost:15672). Ta cũng phải thêm port mapping 5672, default port RabbitMQ sử dụng để comunication.
## c. Setting up RabbitMQ trên Window:
Để cài đặt RabbitMQ trên Windows và có giao diện dashboard management, bạn có thể thực hiện các bước sau:
- Tải RabbitMQ từ trang chủ của RabbitMQ: [tại đây](https://www.rabbitmq.com/download.html). Chọn phiên bản phù hợp với hệ điều hành của bạn.
- Sau khi tải xuống, cài đặt RabbitMQ bằng cách chạy tệp cài đặt đã tải về.
- Sau khi cài đặt xong, bạn cần cấu hình plugin để có thể truy cập vào giao diện dashboard management. Mở Command Prompt với quyền quản trị và chạy các lệnh sau:
-- `cd "C:\Program Files\RabbitMQ Server\rabbitmq_server-{version}\sbin"`
-- `rabbitmq-plugins.bat enable rabbitmq_management`
- Khởi động lại RabbitMQ Server để plugin có hiệu lực. Bạn có thể khởi động lại RabbitMQ bằng cách mở Command Prompt với quyền quản trị và chạy lệnh sau:
`net stop RabbitMQ && net start RabbitMQ`
- Truy cập vào giao diện dashboard management bằng cách mở trình duyệt và truy cập vào địa chỉ http://localhost:15672/. Đăng nhập bằng tài khoản mặc định là "guest" và mật khẩu là "guest".

# Lưu ý:
Để sử dụng port khác thay vì sử dụng mặc định http://localhost:15672/ trên RabbitMQ, bạn có thể thực hiện theo các bước sau:
1. Mở file **rabbitmq.config**, file này thường nằm trong thư mục cài đặt RabbitMQ, ví dụ `C:\Program Files\RabbitMQ Server\rabbitmq_server-{version}\etc\rabbitmq`.
2. Nếu file **rabbitmq.config** chưa tồn tại, hãy tạo một file mới có tên là **rabbitmq.config**.
3. Thêm dòng mã sau vào file **rabbitmq.config** để cấu hình port mới:
`[  
	{rabbitmq_management, [{listener, [{port, 8080}]}]}
]`

Trong đó, số **8080** là port mới mà bạn muốn sử dụng.
4. Lưu file rabbitmq.config và khởi động lại RabbitMQ để cập nhật các thay đổi.

