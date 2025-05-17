# Hướng dẫn sử dụng hàm `print` với `PrintItem` (Sử dụng String cho Type và Align)

Để thực hiện lệnh in, bạn cần chuẩn bị một mảng các đối tượng, mỗi đối tượng mô tả một thành phần cần in. Chúng ta gọi mỗi đối tượng này là một `PrintItem`.

## Các giá trị `type` được hỗ trợ (dưới dạng chuỗi):

*   `'text'`: In văn bản.
*   `'qrCode'`: In mã QR.
*   `'barCode'`: In mã vạch.
*   `'blankLines'`: In một số dòng trống với chiều cao tùy chỉnh.
*   `'feedPaper'`: Đẩy giấy một số dòng theo chiều cao dòng chuẩn.

## Các giá trị `align` được hỗ trợ (dưới dạng chuỗi, tùy chọn):

*   `'left'`: Căn lề trái (thường là mặc định).
*   `'center'`: Căn lề giữa.
*   `'right'`: Căn lề phải.

## Cấu trúc `PrintItem` và các `Options`

Bạn cần định nghĩa các interface TypeScript sau trong project của mình để đảm bảo cấu trúc dữ liệu chính xác:

```typescript
// Định nghĩa các interface Options cho từng loại PrintItem
interface TextPrintOptions {
  fontSize?: 16 | 24 | 32 | 48; // Kích thước font (theo SDK)
}

interface QRCodePrintOptions {
  moduleSize?: number; // Kích thước module (1-8, mặc định 6 theo SDK)
  errorLevel?: 0 | 1 | 2 | 3; // Mức sửa lỗi (0:L, 1:M, 2:Q, 3:H)
}

interface BarCodePrintOptions {
  symbology: number; // Loại mã vạch (tham khảo SDK, ví dụ 8 cho CODE128)
  height?: number; // Chiều cao mã vạch (1-8, mặc định 3 theo SDK)
  width?: number; // Chiều rộng mỗi thanh mã vạch (1-8, mặc định 6 theo SDK)
  textPosition?: 0 | 1 | 2 | 3; // Vị trí văn bản của mã vạch
}

interface BlankLinesPrintOptions {
  lines: number; // Số dòng trống (tối đa 100 theo SDK)
  heightPerLine: number; // Chiều cao của mỗi dòng trống (đơn vị: Pixel)
}

interface FeedPaperPrintOptions {
  lines: number; // Số dòng giấy cần đẩy
}

// Định nghĩa kiểu cho giá trị căn lề
type PrintItemAlign = 'left' | 'center' | 'right';

// Định nghĩa kiểu chính cho một mục cần in
type PrintItem =
  | { type: 'text'; content: string; align?: PrintItemAlign; options?: TextPrintOptions; }
  | { type: 'qrCode'; content: string; align?: PrintItemAlign; options?: QRCodePrintOptions; }
  | { type: 'barCode'; content: string; align?: PrintItemAlign; options: BarCodePrintOptions; } // `options.symbology` là bắt buộc
  | { type: 'blankLines'; options: BlankLinesPrintOptions; content?: never; align?: never; }
  | { type: 'feedPaper'; options: FeedPaperPrintOptions; content?: never; align?: never; };

// Hàm print bạn sẽ tự triển khai, ví dụ:
// async function print(items: PrintItem[], feedAfterPrint?: number): Promise<void> {
//   // Logic của bạn để kết nối, gửi lệnh in tuần tự, và gọi performPrint sẽ ở đây
// } 
```

## Cách tạo các `PrintItem` cụ thể:

### 1. In Văn bản (type: `'text'`)
*   **`type`**: `'text'`
*   **`content`**: Chuỗi văn bản cần in.
*   **`align`** (tùy chọn): `'left'`, `'center'`, `'right'`.
*   **`options`** (tùy chọn) `TextPrintOptions`:
    *   `fontSize`: `16 | 24 | 32 | 48`.

**Ví dụ:**
```typescript
const textItem1: PrintItem = {
  type: 'text',
  content: "HÓA ĐƠN THANH TOÁN",
  align: 'center',
  options: { fontSize: 32 }
};

const textItem2: PrintItem = {
  type: 'text',
  content: "Sản phẩm A: 2 x 50.000",
  // align sẽ là mặc định (ví dụ: 'left') nếu không được chỉ định
  // fontSize sẽ là mặc định của máy in nếu không được chỉ định
};
```

### 2. In Mã QR (type: `'qrCode'`)
*   **`type`**: `'qrCode'`
*   **`content`**: Dữ liệu (chuỗi) của mã QR.
*   **`align`** (tùy chọn): `'left'`, `'center'` (thường là mặc định), `'right'`.
*   **`options`** (tùy chọn) `QRCodePrintOptions`:
    *   `moduleSize`: Kích thước module (1-8, mặc định 6).
    *   `errorLevel`: Mức sửa lỗi (0-L, 1-M, 2-Q (mặc định), 3-H).

**Ví dụ:**
```typescript
const qrCodeItem: PrintItem = {
  type: 'qrCode',
  content: "https://yourstore.com/payment/12345",
  align: 'center',
  options: { moduleSize: 6, errorLevel: 2 } // errorLevel 2 tương ứng mức 'Q'
};
```

### 3. In Mã Vạch (type: `'barCode'`)
*   **`type`**: `'barCode'`
*   **`content`**: Dữ liệu (chuỗi) của mã vạch.
*   **`align`** (tùy chọn): `'left'`, `'center'` (thường là mặc định), `'right'`.
*   **`options`** (bắt buộc) `BarCodePrintOptions`:
    *   `symbology`: Số nguyên đại diện cho loại mã vạch (tham khảo tài liệu SDK, ví dụ `8` cho CODE128).
    *   `height` (tùy chọn): Chiều cao (1-8, mặc định 3).
    *   `width` (tùy chọn): Độ rộng mỗi thanh (1-8, mặc định 6).
    *   `textPosition` (tùy chọn): Vị trí text (`0`: không, `1`: trên, `2`: dưới (mặc định), `3`: cả hai).

**Ví dụ:**
```typescript
const barCodeItem: PrintItem = {
  type: 'barCode',
  content: "PRODUCT001",
  align: 'center',
  options: {
    symbology: 8, // Ví dụ: CODE128
    height: 5,
    width: 2,
    textPosition: 2 // Hiển thị text ở dưới mã vạch
  }
};
```

### 4. In Dòng Trống (type: `'blankLines'`)
*   **`type`**: `'blankLines'`
*   **`options`** (bắt buộc) `BlankLinesPrintOptions`:
    *   `lines`: Số lượng dòng trống (tối đa 100 theo SDK).
    *   `heightPerLine`: Chiều cao (pixel) của mỗi dòng trống.
*   `content` và `align` không áp dụng cho loại này.

**Ví dụ:**
```typescript
const blankLinesItem: PrintItem = {
  type: 'blankLines',
  options: { lines: 3, heightPerLine: 16 } // 3 dòng trống, mỗi dòng cao 16 pixel
};
```

### 5. Đẩy Giấy (type: `'feedPaper'`)
*   **`type`**: `'feedPaper'`
*   **`options`** (bắt buộc) `FeedPaperPrintOptions`:
    *   `lines`: Số dòng giấy cần đẩy.
*   `content` và `align` không áp dụng cho loại này.

**Ví dụ:**
```typescript
const feedPaperItem: PrintItem = {
  type: 'feedPaper',
  options: { lines: 5 } // Đẩy giấy ra 5 dòng chuẩn
};
```

## Cách gọi hàm `print` (Ví dụ tổng hợp)

Sau khi chuẩn bị một mảng các `PrintItem`, bạn sẽ truyền nó vào hàm `print` mà bạn đã định nghĩa trong logic ứng dụng của mình.

```typescript
// Giả sử các interface và type PrintItem đã được định nghĩa ở trên.

const receiptData: PrintItem[] = [
  { type: 'text', content: "TÊN CỬA HÀNG", align: 'center', options: { fontSize: 32 } },
  { type: 'text', content: "Địa chỉ: 123 Đường ABC, Quận XYZ, TP HCM", align: 'center', options: { fontSize: 16 } },
  { type: 'text', content: "SĐT: 0909123456", align: 'center', options: { fontSize: 16 } },
  { type: 'blankLines', options: { lines: 1, heightPerLine: 24 } },
  { type: 'text', content: "HÓA ĐƠN TÍNH TIỀN", align: 'center', options: { fontSize: 24 } },
  { type: 'text', content: `Ngày: ${new Date().toLocaleDateString('vi-VN')}`, align: 'left' },
  { type: 'text', content: "--------------------------------", align: 'center' },
  { type: 'text', content: "Sản phẩm 1       SL: 1   100.000", align: 'left' },
  { type: 'text', content: "Sản phẩm 2 dài   SL: 2   200.000", align: 'left' },
  { type: 'text', content: "--------------------------------", align: 'center' },
  { type: 'text', content: "Tổng cộng:               300.000", align: 'right', options: { fontSize: 24 } },
  { type: 'blankLines', options: { lines: 1, heightPerLine: 30 } },
  { type: 'barCode', content: "HD00120231027", align: 'center', options: { symbology: 8, height: 4, width: 2, textPosition: 0 } },
  { type: 'blankLines', options: { lines: 1, heightPerLine: 10 } },
  { type: 'qrCode', content: "Cảm ơn quý khách!", align: 'center', options: { moduleSize: 4 } },
  { type: 'text', content: "Hẹn gặp lại!", align: 'center', options: { fontSize: 16 } },
  { type: 'feedPaper', options: { lines: 5 } } // Đẩy giấy ở cuối để dễ xé
];

// Gọi hàm print bạn đã tự viết với dữ liệu trên
// p.print(receiptData, 160) // 160 là số dòng feed sau khi in (ví dụ)
//   .then(() => {
//     console.log("Hóa đơn đã được gửi để in!");
//     // Thông báo cho người dùng
//   })
//   .catch(error => {
//     console.error("Lỗi khi thực hiện in:", error);
//     // Thông báo lỗi cho người dùng
//   });
```

**Lưu ý quan trọng khi bạn tự triển khai hàm `print`:**

*   Hàm `print` của bạn cần lặp qua mảng `items`.
*   Với mỗi `item`, sử dụng `switch (item.type)` để xác định hành động:
    *   Nếu `item.type` là `'text'`, `'qrCode'`, hoặc `'barCode'`, bạn cần gọi hàm `setAlignment` (nếu `item.align` được cung cấp) và các hàm cài đặt `options` (như `setFontSize`) trước khi gọi hàm in nội dung (`printText`, `printQRCode`, `printBarCode`).
    *   Nếu `item.type` là `'blankLines'` hoặc `'feedPaper'`, bạn gọi trực tiếp hàm tương ứng với các `options` đã cho.
*   **Sau khi xử lý tất cả các `item` trong mảng, hàm `print` của bạn phải gọi `PosPrinterAPI.performPrint(feedAfterPrint)` một lần duy nhất để máy in thực thi tất cả các lệnh đã được gửi vào bộ đệm.**

Hướng dẫn này cung cấp cách để người dùng chuẩn bị dữ liệu một cách rõ ràng và có cấu trúc để sử dụng hàm `print` mà bạn sẽ xây dựng.
```
