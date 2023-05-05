**Contact Me:** https://t.me/DontBeSad_Cuz_MinhFatIsHere

## Lưu ý 

Liên hệ để mua mã nguồn. Đây là bộ server xử lý logic bot dùng cho phía nhà cái muốn có bot để tiếp user.

**Đầu vào là toàn bộ trạng thái của trò chơi, các lá bài của mọi người, lá bài dưới nọc và đầu ra là lựa chọn tối ưu nhất.**

**Vì thế nên đây sẽ là Bot nhìn bài, tuy nhiên sẽ đánh giống 1 người bình thường và có thể điều chỉnh trình độ từ cơ bản tới cao thủ trên từng lượt đánh  và đôi khi có những sáng tạo để tạo ra đột biến trong ván chơi.**

**Bot hỗ trợ nhiều phương thức để điều chỉnh độ mạnh yếu giúp vận hành tiền hồ cho nhà cái**

## Cấu trúc

Document cho API Tiến lên Bot Service.


**Các tính năng chính:**
- Chịu tải lớn, nhanh, độ trễ thấp, có thể chạy đồng thời hàng nghìn con bot.
- Chơi giống hệt người, không thể phân biệt được đâu là Bot đâu là người.
- Điều chỉnh độ thông minh một cách linh hoạt, giúp vận hành bot một cách dễ dàng nhất.
- Deploy sử dựng Docker.
- Kích thước nhỏ nhẹ, toàn bộ source chỉ vài MB.

Cấu trúc Bot:

| Cấu trúc hệ thống                                                                                                                                                                                                                                                                | 
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **API Service:** Server nhận request từ bên ngoài                                                                                                                                                                                                                                |
| **Input Parser:** thực hiện chuyển yêu cầu từ bên ngoài thành input cho bot.                                                                                                                                                                                                     |
| **Optimizer:** từ những quyết định có thể của bot, tối ưu hóa lược bớt các quyết định không giống người hoặc gây mất lợi thế                                                                                                                                                     |
| **Bot Algorithm:** thuật toán nhận đầu vào từ Optimizer và thực hiện tính toán để đưa ra quyết định tốt nhất trong những quyết định đó                                                                                                                                           |
| **Output Parser:** chuyển output của bot thành output mà người call API mong muốn.                                                                                                                                                                                               |
| Service được deploy bằng **docker** giúp tối ưu hóa thời gian triển khai hệ thống, hệ thống ghi log chi tiết từng request theo ngày. Service chạy ổn định và tự động khởi động lại nếu gặp lỗi, chịu tải số lượng lớn hàng trăm nghìn bot một lúc cho một máy chủ cấu hình thấp. |


_BaseURL_: http://localhost:9701

## API Ping

```http request
GET /api/v2/ping
Content-Type: application/json
```

_Response_

| Parameter | Type     |
|:----------|:---------|
| `status`  | `string` |

## API Select Best Selection

```http request
POST /api/v2/best-selection
Authorization: "Bearer Key"
Content-Type: application/json
```

_**Body:**_
- _players_ `Player[]` danh sách người chơi trên bàn., theo thứ tự của vòng chơi ví dụ trên bàn còn 4 người chơi A, B, C, D, đang lượt chơi của C và tiếp theo sẽ là D, A, B theo thứ tự. thì mảng này sẽ sắp xếp theo thứ tự [A, B, C, D], hoặc [B, C, D, A] hoặc [C, D, A, B], ..etc. miễn sao là theo thứ tự của vòng là được.
- _previous_player_id_ `string` id của người chơi trước liền kề người chơi hiện tại (**không tính người pass**) của round hiện tại, 
  + **Nếu lượt hiện tại là là lượt đầu của ván hoặc round hiện tại thì đặt thành chuỗi rỗng.** 
  + A đánh, B đánh đến C thì previous player id = B
  + A đánh, B pass, đến lượt C thì previous player id = A.
- _current_player_id_ `string` id của người chơi hiện tại, không được rỗng hoặc null.
- _combination_ `Combination` bộ bài **gần nhất** được đánh trên bàn (**không thể là Pass**), ví dụ:
  + Nếu A đánh đôi 2 rồi đến lượt B thì combination là đôi 2
  + Nếu B Pass rồi đến C thì combination **vẫn là đôi 2**
  + **Còn nếu là lượt đầu của round mới hoặc game mới thì để là null**.
- _is_first_play_ `bool` true nếu lượt này bắt buộc phải đánh 3 bích (tuỳ vào luật, thường là lượt chơi đầu của game đầu trong bàn).
- _allow_first_play_cards_ `int[]` danh sách các lá bài bắt buộc phải đánh (hoặc các bộ chứa 1 trong các lá đò) nếu `is_first_play = true`, nếu người chơi không có lá nào trong các lá này thì sẽ được đánh thoải mái không bắt buộc gì, Còn trong trường hợp không gửi lên hoặc rỗng thì mặc định sẽ bắt buộc phải đánh lá nhỏ nhất trong bộ, hoặc 1 bộ có chứa lá đó. (ví dụ [3♠] thì tức là bắt buộc phải đánh là 3♠ hoặc bộ chứa 3♠ trước, nếu không có 3♠ thì có thể đánh bình thường)
- _passed_can_play_special_combination_ `bool` đặt là true nếu có thêm luật: 3,4 đôi thông hoặc tứ quý có thể chặt 2 hoặc đôi 2 kể cả khi đã mất lượt.
- _config_ `Config` config chi tiết cho thuật toán.

_**Response:**_
- _status_code_ `int` mã code trạng thái trả về, `Success = 0, Error != 0`.
- _status_desc_ `string` mô tả lỗi.
- _selection_ `Combination` bộ bài mà bot chọn để đánh ra.

_Player:_
- _id_ `string` định danh của người chơi.
- _passed_ `bool` true nếu mà player đã pass trước đó, tức là không còn được chơi tiếp tại round này. Nếu round này vẫn được chơi thì passed = false. Ví dụ A đánh 3 bích, B pass, C đánh 4 cơ, A pass thì đến lượt C tức là bắt đầu round mới thì passed của A,B đều là false.
- _is_bot_ `bool` chỉ định người này là bot.
- _cards_ `integer[]` danh sách các lá bài trên tay người này.

_Combination:_
- _cards_ `integer[]` danh sách lá bài của bộ kết hợp.
- _type_ `integer` loại của bộ (xem mô tả bên dưới).

_Config:_
- _interactions_ `int` số lượng vòng loop trong thuật toán, càng ít càng chơi kém, default: 1.000.000.000 loops (nếu gửi lên bằng 0 hoặc không gửi thì sẽ được set là default).
- _min_thinking_time_ `long` số milliseconds tối thiểu dành cho bot, tức là bot sẽ dành ít nhất bằng này thời gian để suy nghĩ cho mỗi request, không bao giờ nghĩ ít hơn (nếu gửi lên bằng 0 hoặc không gửi thì sẽ được set là default 500).
- _max_thinking_time_ `long` số milliseconds tối đa danh cho bot, tức là bot sẽ dành nhiều nhất bằng này thời gian để suy  nghĩ cho mỗi yêu cầu, không bao giờ suy nghĩ lâu hơn (nếu gửi lên bằng 0 hoặc không gửi thì sẽ được set là default 1000).

## Convert Card to Integer Number

Các giá trị của _Rank_ và _Suit_ 

| Rank  | Default Value |
|-------|---------------|
| Two   | 0             |
| Three | 1             |
| Four  | 2             |
| Five  | 3             |
| Six   | 4             |
| Seven | 5             |
| Eight | 6             |
| Nine  | 7             |
| Ten   | 8             |
| Jack  | 9             |
| Queen | 10            |
| King  | 11            |
| Ace   | 12            |

| Suit    | Default Value |
|---------|---------------|
| Spade   | 0             |
| Club    | 1             |
| Diamond | 2             |
| Heart   | 3             |

| Công thức chuyển đổi                                                                                  |
|-------------------------------------------------------------------------------------------------------|
| **card value = rank * 4 + suit**<br/>**suit = card value % 4**<br/>**rank = (card value - suit) / 4** |

## Combination Type

| Type                    | Value | Note        |
|-------------------------|-------|-------------|
| Single                  | 0     | Lá đơn      |
| Pair                    | 1     | Đôi         |
| Three Of A Kind         | 2     | Bộ Tam      |
| Four Of A Kind          | 3     | Tứ Quý      |
| Straight                | 4     | Sảnh        |
| Two Consecutive Pairs   | 5     | 2 Đôi Thông |
| Three Consecutive Pairs | 6     | 3 Đôi Thông |
| Four Consecutive Pairs  | 7     | 4 Đôi Thông |
| Pass                    | 8     | Bỏ Lượt     |

## Status Code

| Code | Description                                                                                           |
|------|-------------------------------------------------------------------------------------------------------|
| 1    | Body trong request gửi sai định dạng                                                                  |
| 2    | ID người chơi không thể rỗng hoặc null                                                                |
| 3    | giá trị card phải chạy từ 0 tới 51.                                                                   |
| 4    | Không tìm thấy người chơi trong Players List                                                          |
| 5    | Mọi người đều pass là vô lý                                                                           |
| 6    | Không tìm thấy previous player ID trong Players List (trong trường hợp previous_player_id được gán)   |
| 7    | Không tìm thấy current player ID trong Players List                                                   |
| 8    | trong trường hợp không phải first turn của round này thì combination không được rỗng.                 |
| 9    | người chơi hiện tại không thể pass ở turn trước được.                                                 |
| 10   | combination type phải nằm trong khoảng từ 0 đến 8 và phải khớp với các lá bài trong bộ đó.            |
| 11   | thời gian tối thiểu suy nghĩ của bot luôn luôn phải nhỏ hơn hoặc bằng thời gian tối đa suy nghĩ.      |
| 12   | nếu là turn đầu trong round đầu của bàn thì bắt buộc người chơi hiện tại phải có lá 3 bích trong tay. |
| 13   | Có 2 người chơi trùng ID với nhau                                                                     |
| 14   | Có 2 lá bài trong tay mọi người chơi và trên bàn trùng nhau.                                          |
| 15   | quá thời gian suy nghĩ                                                                                |

## Test

```shell
curl -X POST --location "http://localhost:9701/api/v1/tienlen/bestselection" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer abcXYZ123456789" \
    -d "{
          \"players\": [
            { \"id\": \"A\", \"passed\": false, \"is_bot\": false, \"cards\": [ 2, 3,4 ] },
            { \"id\": \"B\", \"passed\": true, \"is_bot\": false, \"cards\": [ 5, 6,7,8 ]},
            { \"id\": \"C\", \"passed\": false, \"is_bot\": true, \"cards\": [ 9,10,11,12 ] },
            { \"id\": \"D\", \"passed\": false, \"cards\": [ 13, 14,15 ] }
          ],
          \"previous_player_id\": \"A\",
          \"current_player_id\": \"C\",
          \"combination\": {\"cards\": [1],\"type\": 0 },
          \"config\": {
            \"interactions\": 1000000,
            \"min_thinking_time\": 500,
            \"max_thinking_time\": 2000,
            \"force\": false
          }
        }"
```
