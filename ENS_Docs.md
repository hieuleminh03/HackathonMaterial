## Định nghĩa:
- ENS =  Ethereum Name Service. dựa trên Ethereum. mục tiêu giống như DNS của internet
- ENS có vai trò nối các tên miền như hieuminhle.eth với các dữ liệu như địa chỉ ví Ethereum, các nội dung hash, siêu dữ liệu, ... 
- ENS hỗ trợ reverse resolution: cho phép liên kết những metadata như tên hoặc interface description với địa chỉ ví.

## Các thành phần trong ENS:

![Component](/assets/component.png)

- Registry (kho lưu trữ): lõi smart contract chứa danh sách các tên miền và tên miền phụ nối với các địa chỉ ví. Khi người dùng tạo mới một tên miền, contract sẽ gắn nó với địa chỉ ethereum của họ. 
- Resolver (trình phân giải): cho phép cập nhật tên miền thành địa chỉ ví. resolver sẽ gọi đến registry để xử lý các truy vấn.
- ENS Name: Chính là tên miền.

## Quy trình resolve của ENS:
- Quá trình lấy dữ liệu từ một tên miền gọi là quá trình phân giải (resolution):
![Resolution](/assets/flow1.png)
- Các bước lần lượt như sau:
    1. Tìm kiếm Resolver:
        - Mỗi tên miền đều có một resolver. Resolver chỉ đơn giản là một contract để query data của một tên miền.
            ```javascript
            const resolver = await provider.getResolver("luc.eth");
            ```
    2. Truy vấn vào Resolver:
        - Call đến các function mà resolver cung cấp. Ví dụ: `addr()`, `content()`, `interfaceID()`, ...
            ```javascript
            const addr = await resolver.addr("luc.eth");
            ```

## Quy trình xử lý tên miền:
- Hầu hết các thư viện đã có các lớp xử lý. Khi làm dApp chưa cần quá quan tâm.
- Đọc thêm tai [đây](https://docs.ens.domains/resolution/names) 

## Phân tích cụ thể các thành phần của ENS:
	a. Registry: 
		- Registry là lõi contract của hệ thống.
		- Các TLD (Top Level Domain) được sở hữu bởi các registrar
		
		- Bao gồm 2 contract: BaseRegistrar và ETHRegistrarController (gọi là A và B)
		- A có trách nhiệm cho việc sở hữu tên miền, chuyển giao,... (các nghiệp vụ liên quan đến sở hữu), B xử lý đăng ký & gia hạn (liên quan đến tính tiền & giá). B được coi là phần chính của 