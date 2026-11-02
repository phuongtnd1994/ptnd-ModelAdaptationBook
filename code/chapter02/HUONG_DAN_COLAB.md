# Hướng dẫn chạy Chương 2: How to do model adaptation trên Google Colab

Notebook: [`chapter02_colab.ipynb`](chapter02_colab.ipynb)

## Mục tiêu

Chạy quickstart fine-tune LoRA 5 bước trên Qwen3-4B (40 ví dụ IT-support), gọi thử Claude API (Listing 2.1) và so sánh với adapter chương 5.

## Yêu cầu

| Hạng mục | Chi tiết |
|---|---|
| GPU | **L4 hoặc A100** (bước train dùng bf16 — T4 không chạy được) |
| Thời gian | ~15–20 phút (bao gồm tải model ~8 GB) |
| API | Listing 2.1 dùng `ANTHROPIC_API_KEY` (1 lời gọi). |

## Cách mở notebook trên Google Colab

**Cách 1 – Mở trực tiếp từ GitHub (khuyến nghị):**
1. Push repo lên GitHub (repo `phuongtnd1994/ptnd-ModelAdaptationBook` phải ở chế độ *public*, hoặc bạn đăng nhập GitHub trong Colab để mở repo private).
2. Vào <https://colab.research.google.com> → **File › Open notebook › GitHub** → dán `phuongtnd1994/ptnd-ModelAdaptationBook` → chọn file `code/chapter02/chapter02_colab.ipynb`.
   Hoặc bấm badge **Open in Colab** ở đầu notebook.

**Cách 2 – Upload file:** **File › Upload notebook** → chọn `chapter02_colab.ipynb` từ máy.

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
| 0. Cài đặt | Clone repo, cài package, kiểm tra GPU, nạp API key. |
| 1. Listing 2.1 | Gọi `claude-opus-5-5` một lần — minh hoạ phương án "buy". |
| 2. Dữ liệu | Kiểm tra dữ liệu IT-support đã commit sẵn (không cần build lại). |
| 3. Quickstart | `python -m chapter02.quickstart` — train LoRA 20 bước, in output trước/sau, lưu adapter. |
| 4. Preview ch5 | `python -m chapter02.run_chapter5_adapter` — so sánh base vs adapter chương 5 tải từ Hub. |
| 5. Lưu Drive | Copy `chapter02/runs` sang Google Drive (tuỳ chọn). |

## Kết quả mong đợi

Log `Step 1..5`, loss giảm trong 20 bước, peak GPU ~9 GB. Adapter (~130 MB) + `manifest.json` nằm trong `chapter02/runs/ch2_quickstart/`.

## Lỗi thường gặp

| Triệu chứng | Cách xử lý |
|---|---|
| `bf16`/`Attempting to unscale FP16 gradients` | Đang dùng T4 → đổi runtime sang L4/A100. |
| `CUDA out of memory` | Giảm `max_length` từ 512 xuống 256 trong `chapter02/quickstart.py`. |
| `No module named 'chapterXX'` | Chạy lại ô cài đặt đầu tiên (nó `cd` vào `code/` và thêm vào `sys.path`). |
| Mất kết nối / runtime reset | Chạy lại từ ô cài đặt; lưu kết quả sang Drive sau mỗi bước dài. |

Tài liệu gốc của chương: [README.md](README.md).
