# Góc sinh viên — cập nhật dữ liệu V1

## Nguồn dữ liệu và cấu trúc

`places-source.config.json` là nơi cấu hình duy nhất cho nguồn dữ liệu hiện tại. Để trống `baseUrl` thì app chưa tải dữ liệu thật, người dùng vẫn có thể bấm “Xem dữ liệu mẫu”. Có thể ghi đè URL bằng biến build công khai `EXPO_PUBLIC_MAYUNI_PLACES_URL`. Không đặt token hoặc secret trong URL.

Cấu trúc template trong `student-corner-data/`:

```text
version.json
config/tnut.json
config/tnu-sfl.json
places/tnut/food.json
places/tnut/cafe.json
places/tnut/billiards.json
places/tnut/karaoke.json
places/tnut/fuel.json
places/tnut/pharmacy.json
places/tnut/convenience.json
places/tnut/repair.json
places/tnut/printing.json
places/tnut/housing.json
places/tnut/services.json
places/tnu-sfl/... (cùng cấu trúc)
images/... (bạn tự bổ sung ảnh đã có quyền sử dụng)
```

Tách config/chunk theo trường để không trộn dữ liệu đa trường. `recommended` và `studentBudget` nằm trên từng Place thay cho `featured/featured.json`: không có bản sao địa điểm thứ hai gây lệch giá/giờ. “Cần gấp” cũng lọc cùng những Place này.

Toàn bộ bản ghi có sẵn là MẪU. Địa chỉ, tọa độ, giá, giờ và đánh giá mẫu không phải thông tin địa điểm thực tế. Nút chỉ đường bị tắt với mẫu.

## Kiểm tra JSON trước khi cập nhật

Chạy `npm.cmd run data:validate` trong project. Lệnh kiểm tra manifest/config/chunk, version và ID trùng giữa các nhóm. Mặc định kiểm tra student-corner-data; có thể truyền đường dẫn thư mục khác sau `--`. Không cần đăng nhập trường, không gọi API trường.

## Đưa template lên GitHub

1. Đưa thư mục `student-corner-data` lên repository dữ liệu công khai của bạn (hoặc repository May_Uni hiện có). Không đưa thông tin tài khoản trường, khóa ký hoặc dữ liệu sinh viên vào đây.
2. Khi có dữ liệu thực tế, nhập thông tin đã kiểm tra, đổi `datasetKind` của trường thành `real`, đặt `sample: false` trên các bản ghi thật và bỏ tiền tố `[MẪU]`. Không đổi nhãn để biến một bản mẫu chưa xác minh thành dữ liệu thật.
3. Đặt `baseUrl` trong `places-source.config.json` thành URL RAW HTTPS kết thúc ở thư mục dữ liệu. Ví dụ cấu trúc URL nếu dùng repo hiện tại và đúng nhánh/thư mục: `https://raw.githubusercontent.com/thief-heart/May_Uni/main/student-corner-data/`. Đây là ví dụ cấu hình, chưa được đẩy lên GitHub hay xác nhận tồn tại bởi lượt triển khai này.
4. Build/cài lại app một lần để áp dụng URL mới. Các lần sửa nội dung JSON phía sau chỉ cần cập nhật version, không cần build lại app.
5. Khi thay toàn bộ nguồn URL, cache tách riêng theo URL + trường. Không lấy cache trường khác thay cho trường chưa có dữ liệu.

## Thêm địa điểm

Trong chunk đúng nhóm, thêm phần tử vào `places`. ID phải duy nhất trong trường, dạng chữ thường/số/gạch ngang/gạch dưới, dài tối đa 64 ký tự. Không tái dùng ID của địa điểm khác.

```json
{
  "id": "your-real-place-id",
  "categoryId": "food",
  "name": "Tên địa điểm bạn đã xác minh",
  "description": "Nhận xét thực tế do bạn viết",
  "address": "Địa chỉ đã xác minh",
  "coordinates": null,
  "cover": "",
  "images": [],
  "phone": "",
  "sample": false,
  "verifiedAt": "2026-10-30",
  "recommended": false,
  "studentBudget": true,
  "opening_hours": { "type": "unknown" },
  "prices": [],
  "author_review": {
    "overall": null, "price": null, "quality": null,
    "service": null, "space": null,
    "text": "", "pros": [], "cons": []
  }
}
```

Đây chỉ là khung nhập, không phải địa điểm có thật. Không biết field nào thì để trống/null. Với tọa độ thật dùng `{"latitude": ..., "longitude": ...}`; không đặt `(0,0)` thay cho tọa độ chưa biết. Ảnh dùng HTTPS, ưu tiên WebP thumbnail, không dùng trang Drive HTML thay cho link ảnh trực tiếp. Điện thoại chỉ điền số của cơ sở được công khai hợp lệ. Không cần lưu link Google Maps riêng vì app tạo chỉ đường từ tọa độ.

## Giá và lịch sử giá

Giữ các bản cũ, thêm bản ghi mới vào `price_history` của món/dịch vụ:

```json
{
  "name": "Tên món hoặc dịch vụ",
  "unit": "VNĐ/suất",
  "price_history": [
    { "price": 25000, "recorded_at": "2026-08-12" },
    { "price": 30000, "recorded_at": "2026-10-29" }
  ]
}
```

Ví dụ minh họa. `price` phải là số >=0, ngày hợp lệ dạng YYYY-MM-DD. App chọn ngày ghi nhận gần nhất không ở tương lai. Mỗi món nên có một giá duy nhất trong một ngày. `unit` là chuỗi mở: VNĐ/món, VNĐ/giờ, VNĐ/giờ/phòng, VNĐ/người, VNĐ/tháng... Không cần đổi model để thêm đơn vị mới. Bộ lọc/sắp xếp giá dùng báo giá thấp nhất của Place, không thể so sánh ý nghĩa một suất ăn với một giờ thuê bàn; hãy chọn danh mục trước khi so giá.

## Giờ hoạt động

Mở cả ngày: `{"type":"always"}`. Chưa biết: `{"type":"unknown"}`.

Theo lịch: cấu trúc `days` chứa monday...sunday, mỗi ngày một mảng các khoảng giờ. Mảng rỗng hoặc ngày không khai báo là đóng cửa. Nếu chưa xác minh lịch thì dùng unknown.

```json
{
  "type": "schedule",
  "verifiedAt": "2026-10-30",
  "days": {
    "monday": [{"open":"18:00", "close":"02:00"}],
    "tuesday": [],
    "wednesday": [{"open":"06:00", "close":"11:00"}, {"open":"17:00", "close":"22:00"}],
    "thursday": [], "friday": [], "saturday": [], "sunday": []
  }
}
```

Ví dụ minh họa. Ca 18:00–02:00 thứ Hai vẫn mở đến 02:00 thứ Ba, dù thứ Ba không có ca mở mới. Đúng 02:00 là đóng. `open == close` không được dùng để biểu diễn 24 giờ; dùng always. Logic dùng múi giờ Việt Nam UTC+7, không phụ thuộc múi giờ điện thoại. Phiên bản này chưa có lịch ngoại lệ lễ/Tết, do đó UI luôn ghi “theo giờ đã ghi” và khuyên kiểm tra lại.

## Danh mục, Cần gấp, trường mới

Thêm một category vào `config/<school>.json`:

```json
{"id":"laundry","label":"Giặt là","emoji":"🧺"}
```

Thêm `laundry` vào `categoryIds` của chunk đang dùng hoặc thêm chunk mới có `id`, `path`, `version`, `categoryIds`. Đưa Place với categoryId laundry vào chunk đó. UI tự hiện category; không cần sửa code. Nhãn nội dung mới hiển thị nguyên văn nếu chưa có bản dịch.

Thêm lựa chọn Cần gấp trong cùng config:

```json
{"id":"laundry-fast","label":"Giặt gấp","emoji":"🧺","categoryIds":["laundry"]}
```

Một lựa chọn có thể lọc nhiều danh mục. Không có database Cần gấp riêng. Ưu tiên: đang mở > chưa biết giờ > đã đóng; trong mỗi nhóm xét khoảng cách đường thẳng, ngày xác minh gần đây, điểm tác giả rồi giá. Không có quyền vị trí thì dùng mốc trường (nếu có), thiếu cả hai vẫn xem được kết quả nhưng không tính khoảng cách. Cần thuốc chỉ là danh mục nhà thuốc, không phải tư vấn y tế hay bảo đảm có thuốc.

Thêm trường vào version.json và config/places theo cùng schema, sử dụng schoolId đã được app/backend registry nhận biết. Việc thêm trường học để đăng nhập là trách nhiệm registry/adapter hiện có, độc lập với module địa điểm.

## Cập nhật version đúng thứ tự

Ví dụ đang ở `sample-1`, chuyển sang `2026-10-30-01`:

1. Sửa chunk đã đổi; đổi `version` của chunk ở JSON đó.
2. Trong config trường, đổi version của descriptor chunk tương ứng, giữ nguyên version các chunk không đổi. Đổi `config.version`.
3. Upload chunk/config trước; cuối cùng đổi school.version trong version.json cho khớp config.version.
4. App kiểm tra manifest khi mở module và đã qua 15 phút, hoặc khi kéo tải lại/bấm Thử lại. Version không đổi thì không tải lại config hoặc các chunk đã đủ đúng version.
5. GitHub/CDN có thể cập nhật chậm. Version không khớp, HTTP lỗi hoặc JSON lỗi sẽ giữ cache tốt; chờ đồng bộ nguồn rồi tải lại. Không đổi nội dung mà giữ nguyên version nếu muốn app nhận cập nhật.

## Validation, cache, ảnh và giới hạn

- Chỉ HTTPS; không URL chứa tài khoản/mật khẩu; đường dẫn chunk không cho `..`, query hoặc URL ngoại lai. Request JSON bỏ credentials, không gửi token TNUT/May Uni, không đăng nhập GitHub.
- JSON tối đa 2 MiB mỗi request, 8 giây/lần; lỗi mạng/HTTP 429/5xx retry tối đa một lần. JSON sai/schema sai không retry vô hạn.
- Tối đa 40 chunk/config, 1000 Place/chunk, 12 ảnh/Place, 40 món/Place, 120 bản giá/món. Cache dữ liệu chuẩn hóa theo whitelist; không cache raw JSON. Tên/path chunk nên ổn định để tránh để lại cache cũ mồ côi.
- Cache public riêng với vault học tập, dùng AsyncStorage. Vị trí của người dùng chỉ nằm trong RAM của màn hình, không persist/upload; không xin quyền background.
- expo-image cache thumbnail/album theo cơ chế disk+memory. Ảnh đã tải có thể xem offline, ảnh chưa tải thì hiện khung thay thế; không bảo đảm cache ảnh tồn tại vĩnh viễn. Album và danh sách dùng FlatList, không tải toàn bộ album ngay từ đầu.
- ≤30 ngày: mới; 31–180 ngày: có thể thay đổi; >180 ngày: cũ; thiếu/ngày ở tương lai: chưa xác minh. Đây là nhắc kiểm tra lại dữ liệu, không phải chứng nhận cơ sở.
- Không có đánh giá cộng đồng, like, bình luận, upload, báo giá mới hay biểu mẫu đề xuất trong V1. Model authorReview độc lập; sau này thêm communityReview/interaction API như phần riêng, không giả lập số lượt đánh giá.

## Chuyển backend sau này

UI -> usePlaces -> PlacesRepository. Factory duy nhất tại `src/student-corner/client.ts`, hiện tạo StaticPlacesRepository. Khi có backend, implement PlacesRepository trả PlacesSnapshot/Place theo cùng model rồi đổi factory. UI không biết GitHub URL. Không chuyển API TNUT hay phiên trường vào repository địa điểm.

## Build và cập nhật source

Giữ nguyên Android signing/SDK và các module hiện có. Sau khi chép update chạy `npm.cmd ci` (đã thêm expo-location và khai báo trực tiếp expo-image). Không cần Expo account để build APK bằng Gradle đang có. CNG iOS dùng app.json để cấu hình lời giải thích quyền foreground và scheme bản đồ. expo-location Android tự khai báo quyền coarse/fine trong manifest của thư viện; không có quyền background.

Trong C:\MayTNUT:

```powershell
npm.cmd ci
npm.cmd run typecheck
npm.cmd run lint
npm.cmd test
$env:NODE_ENV = "production"
cd android
.\gradlew.bat assembleRelease --no-daemon --max-workers=2
```

Giữ các biến ký release đang dùng. Với iOS theo IOS_BUILD.md; module không yêu cầu backend riêng cho địa điểm V1, nhưng đăng nhập trường trên iOS vẫn cần May Uni Backend. Phải build native mới để dùng dependency vị trí, không chỉ thay JavaScript của một APK cũ.
