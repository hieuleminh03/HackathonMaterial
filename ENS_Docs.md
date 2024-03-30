## Danh mục:
1. [Định nghĩa ENS](#định-nghĩa)
2. [Các thành phần trong ENS](#các-thành-phần-trong-ens)
3. [Quy trình resolve của ENS](#quy-trình-resolve-của-ens)
4. [Quy trình xử lý tên miền](#quy-trình-xử-lý-tên-miền)
5. [Cụ thể vể resolver](#cụ-thể-vể-resolver)
6. [Public Resolver](#public-resolver)
7. [Custom Resolver](#custom-resolver)
8. [Onchain và offchain resolution](#onchain-và-offchain-resolution)
9. [Tương tác với Resolver](#tương-tác-với-resolver)

## Định nghĩa:
- ENS =  Ethereum Name Service. dựa trên Ethereum. mục tiêu giống như DNS của internet
- ENS có vai trò nối các tên miền như hieuminhle.eth với các dữ liệu như địa chỉ ví Ethereum, các nội dung hash, siêu dữ liệu, ... 
- ENS hỗ trợ reverse resolution: cho phép liên kết những metadata như tên hoặc interface description với địa chỉ ví.

## Các thành phần trong ENS:

![Component](/assets/component.png)

- `Registry` (kho lưu trữ): lõi smart contract chứa danh sách các tên miền và tên miền phụ nối với các địa chỉ ví. Khi người dùng tạo mới một tên miền, contract sẽ gắn nó với địa chỉ ethereum của họ. 
- `Resolver` (trình phân giải): cho phép cập nhật tên miền thành địa chỉ ví. resolver sẽ gọi đến registry để xử lý các truy vấn.
- `ENS Name`: Chính là tên miền.

## Quy trình resolve của ENS:
- Quá trình lấy dữ liệu từ một tên miền gọi là quá trình phân giải (resolution):
![Resolution](/assets/flow1.png)
- Các bước lần lượt như sau:
    1. `Tìm kiếm Resolver`:
        - Mỗi tên miền đều có một resolver. Resolver chỉ đơn giản là một contract để query data của một tên miền.
            ```javascript
            const resolver = await provider.getResolver("luc.eth");
            ```
    2. `Truy vấn vào Resolver`:
        - Call đến các function mà resolver cung cấp. Ví dụ: `addr()`, `content()`, `interfaceID()`, ...
            ```javascript
            const addr = await resolver.addr("luc.eth");
            ```

## Quy trình xử lý tên miền:
- Hầu hết các thư viện đã có các lớp xử lý. Khi làm dApp chưa cần quá quan tâm.
- Đọc thêm tai [đây](https://docs.ens.domains/resolution/names) 

## Cụ thể vể resolver:
- Resolver là một contract chứa các function để truy vấn dữ liệu của một tên miền.
- Resolver cơ bản:
    ```javascript
    contract MyResolver {
    function addr(bytes32 node) public view returns (address) {
            return 0x225f137127d9067788314bc7fcc1f36746a3c3B5;
        }
    }
    ```
- [Public Resolver](#public-resolver):
    - Resolver mặc định cho tất cả tên miền gọi là Public Resolver chứa các function cơ bản.
- [Phân giải offchain](#onchain-và-offchain-resolution):
    - Xem ở dưới.

## Public Resolver:
- Là resolver cơ bản mặc định. 
- Cho phép cập nhật các records, thêm quyền, lưu trữ dữ liệu,...
- Các function được hỗ trợ:
    - `EIP-137`: Contract address interface (addr())
    - `EIP-165`: Interface Detection (supportsInterface())
    - `EIP-181`: Reverse Resolution (name())
    - `EIP-205`: ABI Resolution for contracts (ABI())
    - `EIP-619`: SECP256k1 public keys (pubkey())
    - `EIP-634`: Text records (text())
    - `EIP-1577`: Content hash resolution (contenthash())
    - `EIP-2304`: Multicoin support (addr())
- *Note*: Có rất nhiều phiên bản và biến thể của public resolver. Để đảm bảo, luôn gọi supportsInterface() để kiểm tra.

## Custom Resolver:

- `Resolver interface`: là 1 contract định nghĩa interface cho bất kỳ resolver nào.
    ```solidity
    contract MyResolver {
        fn addr(bytes32 node) public view returns (address);
    }
    ```
- Phân giải wildcard: 
    - Đôi khi, một resolver có thể phân giải nhiều tên miền. Ví dụ: `*.luc.eth` hoặc `*.hieuminhle.eth`.
    - Để thực hiện, cần thêm một function `supportsInterface()` để kiểm tra.
    - Ví dụ:
        ```solidity
        contract MyResolver {
            function supportsInterface(bytes4 interfaceID) public pure returns (bool) {
                return interfaceID == this.addr.selector;
            }

            function addr(bytes32 node) public view returns (address) {
                return 0x225f137127d9067788314bc7fcc1f36746a3c3B5;
            }
        }
        ```
- Phân giải offchain: Khi dùng custom resolver, chúng ta có thể triển khai phân giải offchain. Xem ở [dưới](#onchain-và-offchain-resolution).

## Onchain và offchain resolution:
- Onchain resolution: dữ liệu được lưu trữ trực tiếp trên blockchain.
- Offchain resolution: dữ liệu được lưu trữ ngoài blockchain, thông qua các contract resolver.

## Tương tác với Resolver:
1. Kiểm tra các interface được hỗ trợ:
    ```solidity
    function supportsInterface(bytes4 interfaceID) external pure returns (bool)
    ```
2. Cập nhật các record của người dùng:
    - Ví dụ hàm setRecord() với ENSjs v3:
    ```javascript
    import { createWalletClient, custom } from 'viem'
    import { mainnet } from 'viem/chains'
    import { addEnsContracts } from '@ensdomains/ensjs'
    import { setRecords } from '@ensdomains/ensjs/wallet'

    const wallet = createWalletClient({
        chain: addEnsContracts(mainnet),
        transport: custom(window.ethereum),
    })
    const hash = await setRecords(wallet, {
        name: 'ens.eth',
        coins: [
            {
            coin: 'ETH',
            value: '0xFe89cc7aBB2C4183683ab71653C4cdc9B02D44b7',
            },
        ],
        texts: [{ key: 'foo', value: 'bar' }],
        resolverAddress: '0x4976fb03C32e5B8cfe2b6cCB31c09Ba78EBaBa41',
    })
    ```
3. Cập nhật Resolver của người dùng:
- Sử dụng function setResolver() của contract ENSRegistry:
    ```javascript
    import { createWalletClient, custom } from 'viem'
    import { mainnet } from 'viem/chains'
    import { addEnsContracts } from '@ensdomains/ensjs'
    import { setResolver } from '@ensdomains/ensjs/wallet'

    const wallet = createWalletClient({
        chain: addEnsContracts(mainnet),
        transport: custom(window.ethereum),
    })
    const hash = await setResolver(wallet, {
        name: 'ens.eth',
        contract: 'registry',
        resolverAddress: '0x4976fb03C32e5B8cfe2b6cCB31c09Ba78EBaBa41',
    })
    ```