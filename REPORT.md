# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | Lê Nguyễn Trâm Anh |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `2A202602760` |
| GitHub | `itskathy05` |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | `4013 / 388–392` |
| Số seed huấn luyện bạn dùng (`--seeds`) | `5` |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1.241 | 21.5% | BLOCK |
| IDM | 1.507 | 0.1% | baseline |
| EgoMLP | 0.756 | 15.2–36.8% | BLOCK |
| PercMLP | 0.609 | 0.8–2.7% | BLOCK |
| PercMLP+ (sau khi sửa) | 0.805 | 0.3–0.3% | RELEASE |

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

Audit có 992.679 dòng và 9.993 quỹ đạo thô; loại 383 quỹ đạo trùng còn 9.610. `Acc_LV` chỉ tương quan 0,045 với đạo hàm của `Speed_LV`, trong khi `Acc_FAV` tương quan 0,929 với đạo hàm `Speed_FAV`, nên cột gia tốc xe dẫn không đáng tin. Tôi dùng gia tốc suy ra bằng cách làm mượt `Speed_LV` rồi lấy đạo hàm theo thời gian; cách này giảm nhiễu mẫu tốc độ. Phải chia theo quỹ đạo vì các khung liền kề trong cùng một chuyến gần như lặp lại, nếu chia khung thì mẫu gần giống có thể lọt vào cả train và test, làm điểm đánh giá lạc quan. Tập sạch có độ dài 70–109 bước ở 10 Hz.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

Phát lại cùng mã cho 0 đoạn khác nhau; refactor float32 khác bit ở cả 600 đoạn nhưng không đoạn nào vượt dung sai `atol=rtol=0,001`, với sai khác tuyệt đối lớn nhất chỉ 0,0000253 m/s². Đây là sai số làm tròn do thứ tự phép tính/độ chính xác khác nhau, nên dung sai nhỏ theo độ lớn lệnh hợp lý hơn so sánh bit tuyệt đối. AEB làm lệch 8/600 đoạn; mở segment 120 thấy từ frame 82 TTC còn 1,49 giây, gap 5,106 m, lệnh đổi từ −4,44 sang −6,00 m/s², vì ngưỡng AEB là 1,5 giây. Phát nhanh nhất ghi nhận 0 frame trễ/rớt, còn phát thời gian thật có 584 frame trễ và 581 frame rớt; phát nhanh bỏ qua hạn xử lý và tải tính toán nên che lỗi thời gian thực.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

Lưới gồm 60 khoảng cách từ 2,76 đến 59,31 m và 45 tốc độ tiến gần từ 0 đến 10 m/s, tổng 2.700 ô. Theo biên phanh cực đại 9 m/s², bất khả thi là 0,96% ô khi trễ bằng 0 và 5,15% khi trễ 0,6 giây, tăng 4,19 điểm phần trăm; IDM chỉ đâm đúng phần bất khả thi trong phép kiểm chứng này. Log khai thác được 618 ca phanh (6,43% quỹ đạo), nhưng giới hạn phanh người lái ở phân vị 99,9 chỉ là 3,62 m/s². Có 5,22% lưới yêu cầu phanh mạnh hơn mức đó, nên log không đại diện đủ vùng phanh gấp; các ca phanh thật chỉ phủ một phần nhỏ miền kiểm thử và không thay thế được lưới cut-in tổng hợp.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

| Bộ điều khiển | L2 vòng hở (m) / hạng | Va chạm vòng kín (%) / hạng |
|---|---:|---:|
| constant-velocity | 1,241 / 3 | 21,5 / 3 |
| IDM | 1,507 / 4 | 0,14 / 1 |
| EgoMLP | 0,756 / 2 | 15,2–36,8 / 4 |
| PercMLP | 0,609 / 1 | 0,8–2,7 / 2 |

Có bốn cặp đảo hạng: constant-velocity/IDM, constant-velocity/EgoMLP, IDM/EgoMLP và IDM/PercMLP. IDM có L2 xấu nhất vì dự báo quy tắc rằng xe dẫn giữ nguyên tốc độ, không bắt chước chính xác quỹ đạo người lái; nhưng trong vòng kín nó quan sát gap và xe dẫn mỗi bước rồi điều chỉnh hành động, nên tránh va chạm tốt. EgoMLP chỉ nhìn lịch sử tốc độ của xe mình, không biết xe dẫn đang chậm hơn hay khoảng cách bao xa, nên L2 trung bình khá nhưng có thể tích lũy lỗi khi tự lái và đâm nhiều.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

Tỷ lệ va chạm EgoMLP dao động 15,2–36,8% giữa 5 seed, chênh 21,2 điểm phần trăm. Seed tốt nhất là 390: 15,2% va chạm, CI Wilson 95% [13,9%, 16,5%]; seed xấu nhất là 388: 36,8%, CI [35,0%, 38,5%]. Hai khoảng không chồng lấp, nên khác biệt do seed lớn hơn sai số lấy mẫu thể hiện trong các khoảng này. Nếu chỉ chạy seed 390, tôi có thể kết luận quá lạc quan rằng EgoMLP ít va chạm hơn nhiều; seed 388 cho một đánh giá gần gấp 2,4 lần. Vì vậy cần báo dải nhiều seed thay vì chọn một lần chạy thuận lợi.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

IDM phía sau được hiệu chỉnh `T=1,4 s`: khoảng cách Wasserstein với phân bố headway người lái là 0,347, thấp hơn nhiều so với xe không phản ứng (0,882); headway trung vị của người lái là 1,87 s. Ở độ trễ 0,6 s, xe sau phát lại làm IDM bị tính 64,9% va chạm phía sau và 70,1% va chạm bất kỳ; khi xe sau phản ứng, va chạm phía sau về 0% và va chạm bất kỳ còn 5,1% (phía trước). Với PercMLP, tổng va chạm là 82,3% ở thế giới replay nhưng 18,4% ở thế giới phản ứng; phần lớn chênh lệch đến từ xe sau replay không hề phanh khi xe thử giảm tốc. Đây là va chạm giả do giả định tác nhân không phản ứng, không phải lỗi riêng của IDM; đánh giá trong thế giới có phản ứng làm IDM trở lại mức tốt nhất, vì thế kết luận phải xét cả hai thế giới.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

PercMLP bị chặn ở R2 vì cận trên Wilson 3,33% cao hơn giới hạn 1,14%, R3 vì cận trên 15,2% vượt baseline 1,2% cộng 5 điểm, và R4 vì vùng cut-in tránh được nhưng vẫn đâm là 13,26% so với mức cho phép 2%. R5 cũng rớt: dải seed 0,8–2,7% rộng 1,9 điểm, quá ngưỡng 1 điểm; R6 rớt với 18,4% so với baseline phản ứng 5,1% cộng 2 điểm. Nguyên nhân gồm cut-in thiếu trong log huấn luyện và sai số bắt chước tích lũy khi planner tự lái. Bản sửa thêm 20.000 cut-in gắn nhãn IDM rồi dùng hai vòng DAgger để lấy trạng thái PercMLP tự đi vào và gắn nhãn lại từ chính các trạng thái đó; sau sửa cả 7 quy tắc qua, R2 có cận trên 0,64%, R4 còn 0,11%, R5 dải 0,3–0,3%. PercMLP+ học hành vi từ IDM và qua cổng chỉ chứng minh không kém hơn theo các ngưỡng này, không chứng minh tốt hơn hay an toàn để triển khai ngay; cần thử thêm với dữ liệu độc lập, mô phỏng cảm biến và thử nghiệm có kiểm soát.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

Bài lab dùng mô hình một chiều, nên không mô tả chuyển làn, hướng lái, địa hình hay động lực học ngang; hành vi an toàn trong mô hình không đảm bảo an toàn trên đường. Bộ điều khiển nhận gap và tốc độ xe dẫn đã có sẵn, không mô phỏng đầy đủ cảm biến, mất phát hiện, độ trễ đo hay lỗi hiệu chuẩn; nhiễu cố định trong kiểm thử chỉ bao phủ một lát rất hẹp. Xe dẫn trên log được phát lại và không phản ứng với xe thử, còn xe sau replay cũng không phanh, nên tương tác nhiều xe có thể tạo va chạm giả hoặc bỏ qua phản ứng thật. Lưới cut-in và 20.000 ca tổng hợp vẫn có miền tham số và giả định vật lý hữu hạn; kết quả ở đó không bao quát mọi tình huống giao thông. Các con số hiện tại dựa trên log 10 Hz, 5 seed và lưới 2.700 ô với độ trễ 0/0,6 giây, nên cũng chưa bao phủ đủ điều kiện vận hành. Trước khi quyết định phát hành cần kiểm thử phần cứng trong vòng lặp với cảm biến/actuator thật, thêm nhiều tác nhân phản ứng và điều kiện đường/thời tiết, rồi chạy thử kín có giám sát trên các kịch bản độc lập và tiêu chí dừng an toàn.
