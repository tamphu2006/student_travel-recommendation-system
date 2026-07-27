# Student Travel Recommendation System

Các ô `TODO` được giữ lại để học viên tự hoàn thiện; phần tài liệu trong repo tập trung giải thích **vì sao thuật toán hoạt động**, không cung cấp đáp án.

## Học viên sẽ xây dựng gì?

Hệ thống đi từ mô tả điểm đến đến một lịch trình nhiều ngày qua bốn stage:

```text
mô tả + anchor
      │
      ▼
Stage 0: TF-IDF + cosine
      │  8 điểm chủ đề
      ▼
Stage 1: join + kiểm tra + ép kiểu
      │  feature table hoàn chỉnh
      ▼
Stage 2: content-based cosine 13D
      │  danh sách top-K
      ▼
Stage 3: K-Means++ + nearest neighbor
         lịch trình theo ngày và thứ tự ghé thăm
```

| Stage | Câu hỏi chính | Kiến thức trọng tâm | Sản phẩm |
|---|---|---|---|
| 0 | Làm sao biến mô tả thành feature số? | TF-IDF, anchor, cosine, min-max | Ma trận 38 × 8 |
| 1 | Làm sao ghép nhiều nguồn mà không lệch dữ liệu? | ETL, join theo khóa, data contract, type casting | Dataset thống nhất |
| 2 | Điểm đến nào khớp sở thích user? | Content-based filtering, vector 13D, top-K | Danh sách xếp hạng |
| 3 | Chia điểm đến theo ngày và đi theo thứ tự nào? | K-Means++, khoảng cách địa lý, heuristic đường đi | Itinerary nhiều ngày |

## Bản chất của hệ thống

- Đây là **content-based recommender**: profile người dùng được so với feature của điểm đến.
- Đây không phải collaborative filtering vì không có lịch sử rating/click/booking của nhiều người dùng.
- Stage 0 là so khớp từ/cụm từ dựa trên TF-IDF và anchor, không phải semantic embedding hiểu ngữ cảnh như một language model.
- Stage 2 tạo một ranking tương đối; cosine score không phải xác suất hay accuracy.
- Stage 3 dùng heuristic để có lời giải nhanh, không bảo đảm lịch trình tối ưu toàn cục.

Những ranh giới này quan trọng: hiểu đúng hệ thống làm được gì sẽ giúp học viên đánh giá kết quả và đề xuất cải tiến đúng chỗ.

## Cách học với notebook

Học theo thứ tự:

1. [Stage 0 — TF-IDF Topic Scoring](notebook./stage0_tfidf_student.ipynb)
2. [Stage 1 — Build Full Dataset](notebook./stage1_build_dataset_student.ipynb)
3. [Stage 2 — Cosine Similarity Recommendation](notebook./stage2_recommendation_student.ipynb)
4. [Stage 3 — Itinerary Builder](notebook./stage3_itinerary_student.ipynb)

Với mỗi TODO:

1. đọc mục tiêu và dự đoán shape/type của kết quả;
2. tự diễn giải công thức bằng lời;
3. điền một phần nhỏ rồi chạy cell;
4. kiểm tra shape, range và vài ví dụ cụ thể;
5. trả lời phần “Tự kiểm tra” trước khi sang stage sau.

Không nên chỉ xem cell chạy không lỗi. Một pipeline vẫn có thể chạy nhưng ghép sai địa điểm, hiểu ngược chiều feature hoặc tạo ranking vô nghĩa.

## Data contract xuyên suốt

| Nhóm cột | Cột | Kiểu/miền giá trị | Vai trò |
|---|---|---|---|
| Định danh | `place`, `province` | string | join và hiển thị |
| Ngữ nghĩa | `beach` … `photo` | float `[0,1]` | cosine ở Stage 2 |
| Thực tế | `budget`, `family`, `access`, `crowd` | float `[0,1]` | cosine ở Stage 2 |
| Thời gian | `best_months` | string | mã hóa theo query ở Stage 2 |
| Địa lý | `lat`, `lng` | float | chia ngày và xếp tuyến ở Stage 3 |

Quy ước dễ nhầm:

- `budget` cao nghĩa là **tiết kiệm/dễ phù hợp ngân sách**.
- `crowd` cao nghĩa là **đông đúc**, không phải yên tĩnh.
- `best_months` dùng mã `jan` … `dec`, nối bằng `-`, hoặc `all-year`.
- Giá trị `1.0` sau min-max nghĩa là lớn nhất trong tập dữ liệu hiện tại, không phải “phù hợp tuyệt đối 100%”.

## File đầu vào và đường dẫn

Repo hiện chỉ chứa notebook học viên; các CSV/JSON đầu vào cần được cung cấp riêng. Các cell `BASE_DIR` đang giả định notebook được chạy trong cấu trúc stage như sau:

```text
stage0/
├── stage0_descriptions.csv
├── stage0_anchors.csv
└── stage0_tfidf_student.ipynb
stage1/
├── stage1_factual.csv
├── stage1_coordinates.csv
└── stage1_build_dataset_student.ipynb
stage2/
├── stage2_user_profiles.csv
└── stage2_recommendation_student.ipynb
stage3/
├── stage3_config.csv
└── stage3_itinerary_student.ipynb
```

Trong checkout hiện tại, cả bốn notebook đang được lưu chung trong thư mục tên thật là `notebook.` (có dấu chấm ở cuối). Đây là cách lưu source của repo, chưa phải cấu trúc runtime mà các đường dẫn trong notebook đang giả định.

Trước khi chạy, hãy kiểm tra `Path('.')` thực sự trỏ đến stage tương ứng và các output của stage trước nằm đúng nơi stage sau đọc.

> **Lưu ý tên output:** Stage 0 ghi file `stage0_tfidf_scores_rerun.csv`, còn Stage 1 đọc `stage0_tfidf_scores.csv`. Tùy cách tổ chức bài học, file không có hậu tố có thể là dữ liệu chuẩn để đối chiếu; nếu muốn nối trực tiếp kết quả tự làm, cần thống nhất tên/path trước khi chạy Stage 1.

Các thư viện ngoài standard library được notebook sử dụng là `numpy`, `pandas` và `scikit-learn`.

## Bản đồ khái niệm

```text
TF-IDF                 biến văn bản thành vector thưa
cosine similarity      đo độ giống về hướng giữa hai vector
min-max                đổi thang đo theo từng feature
join theo khóa         ghép đúng thực thể dù thứ tự file khác nhau
content-based          gợi ý từ feature item và sở thích khai báo
contextual feature     feature thay đổi theo ngữ cảnh truy vấn, ví dụ tháng
K-Means++              chọn tâm khởi tạo tốt hơn rồi phân cụm theo khoảng cách
nearest neighbor       mỗi bước chọn điểm chưa thăm gần nhất
heuristic              lời giải nhanh/hợp lý nhưng không chắc tối ưu
```

## Các hướng mở rộng sau bài thực hành

- thay TF-IDF bằng word/sentence embedding để bắt đồng nghĩa và ngữ cảnh tốt hơn;
- học trọng số feature từ feedback thay vì coi các chiều có vai trò như nhau;
- thêm Precision@K, Recall@K, NDCG@K, coverage và diversity khi có ground truth;
- dùng hard filter hoặc hậu xử lý nếu mùa là ràng buộc bắt buộc;
- cân bằng số điểm/thời lượng mỗi ngày;
- dùng khoảng cách đường bộ, giờ mở cửa và thời gian tham quan thay cho khoảng cách đường chim bay;
- kết hợp dữ liệu hành vi để tạo hybrid recommender.
