# Hướng dẫn chạy Chương 7: Knowledge distillation trên Google Colab

Notebook: [`chapter07_colab.ipynb`](chapter07_colab.ipynb)

## Mục tiêu

Chưng cất (distill) kiến thức từ model SFT chương 6 (teacher) sang một LoRA student nhỏ gọn, đánh giá student/teacher, kiểm tra safety; bonus so sánh với Claude.

## Yêu cầu

| Hạng mục | Chi tiết |
|---|---|
| GPU | **L4** hoặc A100 (student ~10,5 GB; teacher inference ~10 GB) |
| Thời gian | ~1–1,5 giờ nếu chạy hết (đánh giá 3 chiều chiếm lâu nhất) |
| API | Bonus dùng `ANTHROPIC_API_KEY` (3 lời gọi tới Sonnet 5.5 / Opus 5.5 / Haiku 5.5). |

## Cách mở notebook trên Google Colab

**Cách 1 – Mở trực tiếp từ GitHub (khuyến nghị):**
1. Push repo lên GitHub (repo `phuongtnd1994/ptnd-ModelAdaptationBook` phải ở chế độ *public*, hoặc bạn đăng nhập GitHub trong Colab để mở repo private).
2. Vào <https://colab.research.google.com> → **File › Open notebook › GitHub** → dán `phuongtnd1994/ptnd-ModelAdaptationBook` → chọn file `code/chapter07/chapter07_colab.ipynb`.
   Hoặc bấm badge **Open in Colab** ở đầu notebook.

**Cách 2 – Upload file:** **File › Upload notebook** → chọn `chapter07_colab.ipynb` từ máy.

> Notebook tự `git clone` repo vào `/content/ModelAdaptationBook`. Nếu fork của bạn để private, sửa biến
> `REPO_URL` ở ô cài đặt thành `https://github.com/bahree/ModelAdaptationBook.git` (repo gốc, public).

## Chọn GPU

**Runtime › Change runtime type › Hardware accelerator** → chọn **L4** → Save.

| GPU Colab | VRAM | Dùng cho |
|---|---|---|
| T4 (free) | 16 GB | Chỉ inference / CPU; **không** train được vì sách dùng bf16 |
| L4 (Pro / Pay-as-you-go) | 24 GB | Train LoRA, QLoRA, LoRA-DPO, mọi bước inference |
| A100 (Pro) | 40 GB | Như L4 + full SFT chương 6 |

## Thêm API key (Colab Secrets)

1. Bấm biểu tượng **🔑 Secrets** ở thanh bên trái.
2. **Add new secret**: Name = `ANTHROPIC_API_KEY`, Value = key Claude của bạn (`sk-ant-...`). Bật **Notebook access**.
3. (Tuỳ chọn) thêm `HF_TOKEN` (token đọc từ <https://huggingface.co/settings/tokens>) để tải model nhanh hơn.

Không thêm Secrets thì ô cài đặt sẽ hỏi nhập key bằng ô ẩn (`getpass`). **Đừng** dán key trực tiếp vào code rồi commit.

## Chạy

- Chạy **từng ô theo thứ tự** (Shift+Enter), hoặc **Runtime › Run all**.
- Mọi lệnh được chạy bằng hàm `run("...")` từ thư mục `code/`; log hiện trực tiếp, lỗi sẽ dừng ô ngay.
- Lần đầu tải model `Qwen/Qwen3-4B-Instruct-2507` (~8 GB) mất vài phút; các lần sau dùng cache trong cùng runtime.
- Runtime Colab bị xoá khi ngắt kết nối → dùng ô **Lưu kết quả về Google Drive** ở cuối notebook (nếu có).

## Các bước trong notebook

| Bước | Việc làm |
|---|---|
| 0. Cài đặt | Cài package, tải teacher `ch6-sft` từ Hugging Face. |
| 1. Test | `pytest chapter07/tests/`. |
| 2. Stage 1 | Tuỳ chọn: sinh thử 20 output teacher (dữ liệu 200 prompt đã có sẵn). |
| 3. Stage 2 | Lọc chất lượng → `chapter07/data/distill_ready_colab/`. |
| 4. Stage 3 | Train student LoRA → `chapter07/runs/student_run1/`. |
| 5. Stage 4 | Đánh giá base / teacher / student trên 50 câu test. |
| 6. Stage 5 | Robustness/safety check của student. |
| 7. Inference | `chapter07.generate`. |
| 8. Bonus Claude | Tạo shim `/content/run_with_claude.py` thay OpenRouter bằng Claude, chạy `capture_frontier_comparison`. |

## Kết quả mong đợi

Student đạt ~98% Token F1 của teacher (≈0.165 vs 0.168), cao hơn base (≈0.153).

## Lỗi thường gặp

| Triệu chứng | Cách xử lý |
|---|---|
| Teacher not found | Chạy lại ô tải `ch6-sft` ở bước 0. |
| Quality filter loại quá nhiều | Xem tham số lọc trong `chapter07/scripts/prepare_distillation_data.py`. |
| `No module named 'chapterXX'` | Chạy lại ô cài đặt đầu tiên (nó `cd` vào `code/` và thêm vào `sys.path`). |
| pip báo `dependency conflicts` với `gradio`, `diffusers` | Bỏ qua được — các gói Colab cài sẵn mà sách không dùng. Miễn là ô in ra `Đã cài xong.` |
| Mất kết nối / runtime reset | Chạy lại từ ô cài đặt; lưu kết quả sang Drive sau mỗi bước dài. |

## Hiểu kiến thức của chương

Mở [`DISTILLATION_GIAI_THICH.html`](DISTILLATION_GIAI_THICH.html) bằng trình duyệt (tải file về rồi nhấp đúp) để đọc phần giải thích trực quan: ý tưởng, sơ đồ, giải thích code và cách đọc kết quả của chương.

Tài liệu gốc của chương: [README.md](README.md).
