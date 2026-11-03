# Hướng dẫn chạy Chương 5: LoRA and QLoRA Fine-Tuning trên Google Colab

Notebook: [`chapter05_colab.ipynb`](chapter05_colab.ipynb)

## Mục tiêu

Fine-tune Qwen3-4B bằng LoRA và QLoRA trên dữ liệu IT-support, đánh giá (Exact Match, Token F1, safety regression) và chạy inference với adapter.

## Yêu cầu

| Hạng mục | Chi tiết |
|---|---|
| GPU | **L4 (24 GB)** hoặc A100. LoRA ~9 GB, QLoRA ~5 GB (cần CUDA cho bitsandbytes) |
| Thời gian | LoRA ~15–25 phút · Đánh giá ~10–15 phút · QLoRA (tuỳ chọn) thêm ~30–40 phút |
| API | Không cần API key. |

## Cách mở notebook trên Google Colab

**Cách 1 – Mở trực tiếp từ GitHub (khuyến nghị):**
1. Push repo lên GitHub (repo `phuongtnd1994/ptnd-ModelAdaptationBook` phải ở chế độ *public*, hoặc bạn đăng nhập GitHub trong Colab để mở repo private).
2. Vào <https://colab.research.google.com> → **File › Open notebook › GitHub** → dán `phuongtnd1994/ptnd-ModelAdaptationBook` → chọn file `code/chapter05/chapter05_colab.ipynb`.
   Hoặc bấm badge **Open in Colab** ở đầu notebook.

**Cách 2 – Upload file:** **File › Upload notebook** → chọn `chapter05_colab.ipynb` từ máy.

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
| 0. Cài đặt | Cài `.[dev,qlora]`. |
| 1. Validate | `python chapter05/scripts/validate_chapter05.py`. |
| 2. Train LoRA | `python -m chapter05.train_lora ... --out chapter05/runs/it_lora`. |
| 3. Đánh giá | `listing_5_3_evaluate` → `chapter05/runs/eval_report/report.md` (hiển thị ngay trong notebook). |
| 4. Inference | `chapter05.generate` với adapter vừa train. |
| 5. QLoRA | Tuỳ chọn: train 4-bit rồi so sánh 3 chiều base / LoRA / QLoRA. |
| 6. Không train | Dùng adapter `bahree/ModelAdaptationBook#ch5-lora` từ Hub. |

## Kết quả mong đợi

Loss giảm qua 3 epoch; adapter (~130 MB) trong `chapter05/runs/it_lora/`. Token F1 của adapter cao hơn base trên 50 câu test; safety check không tụt.

## Lỗi thường gặp

| Triệu chứng | Cách xử lý |
|---|---|
| Đang dùng T4 | Đổi sang L4/A100 — script tự chọn fp16 trên T4 nhưng model load ở bf16 dễ lỗi. |
| `Some parameters are on the meta device` | GPU hết bộ nhớ và đang offload sang CPU → restart runtime để giải phóng VRAM. |
| Runtime ngắt giữa chừng | Lưu `chapter05/runs` sang Drive sau mỗi bước dài. |
| `No module named 'chapterXX'` | Chạy lại ô cài đặt đầu tiên (nó `cd` vào `code/` và thêm vào `sys.path`). |
| pip báo `dependency conflicts` với `gradio`, `diffusers` | Bỏ qua được — các gói Colab cài sẵn mà sách không dùng. Miễn là ô in ra `Đã cài xong.` |
| Mất kết nối / runtime reset | Chạy lại từ ô cài đặt; lưu kết quả sang Drive sau mỗi bước dài. |

## Hiểu kiến thức của chương

Mở [`QLORA_GIAI_THICH.html`](QLORA_GIAI_THICH.html) bằng trình duyệt (tải file về rồi nhấp đúp) để đọc phần giải thích trực quan: ý tưởng, sơ đồ, giải thích code và cách đọc kết quả của chương.

Tài liệu gốc của chương: [README.md](README.md).
