# Hướng dẫn chạy Chương 6: Supervised Fine-Tuning (SFT) trên Google Colab

Notebook: [`chapter06_colab.ipynb`](chapter06_colab.ipynb)

## Mục tiêu

Hiểu full SFT: chạy inference, đánh giá, behavioral tests và safety regression trên model SFT; tuỳ chọn tự train full SFT nếu có A100.

## Yêu cầu

| Hạng mục | Chi tiết |
|---|---|
| GPU | Đường A: **L4** trở lên (inference ~8–10 GB). Đường B (train): **A100 40 GB** hoặc H100 + ~80 GB ổ đĩa |
| Thời gian | Đường A: ~20–30 phút · Đường B: thêm ~45–60 phút train |
| API | Không cần API key. |

## Cách mở notebook trên Google Colab

**Cách 1 – Mở trực tiếp từ GitHub (khuyến nghị):**
1. Push repo lên GitHub (repo `phuongtnd1994/ptnd-ModelAdaptationBook` phải ở chế độ *public*, hoặc bạn đăng nhập GitHub trong Colab để mở repo private).
2. Vào <https://colab.research.google.com> → **File › Open notebook › GitHub** → dán `phuongtnd1994/ptnd-ModelAdaptationBook` → chọn file `code/chapter06/chapter06_colab.ipynb`.
   Hoặc bấm badge **Open in Colab** ở đầu notebook.

**Cách 2 – Upload file:** **File › Upload notebook** → chọn `chapter06_colab.ipynb` từ máy.

> Notebook tự `git clone` repo vào `/content/ModelAdaptationBook`. Nếu fork của bạn để private, sửa biến
> `REPO_URL` ở ô cài đặt thành `https://github.com/bahree/ModelAdaptationBook.git` (repo gốc, public).

## Chọn GPU

**Runtime › Change runtime type › Hardware accelerator** → chọn **A100** → Save.

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
| 0. Cài đặt | Cài package, tải `ch6-sft` (~8 GB) từ Hugging Face về `/content/hub/ch6-sft`. |
| A1. Generate | `chapter06.generate` với câu hỏi VPN. |
| A2. Eval | `chapter06.eval_sft` — Token F1 base vs SFT trên 50 câu test. |
| A3. Behavioral | `behavioral_tests --also_test_base`. |
| A4. Safety | `safety_regression` → `chapter06/eval/safety_colab/`. |
| B. Train | Kiểm tra VRAM ≥ 40 GB → smoke test 2 bước → train đủ 3 epoch → monitor checkpoint. |

## Kết quả mong đợi

Token F1 tổng thể: base ≈ 0.153, SFT ≈ 0.168. Safety regression không tụt so với base.

## Lỗi thường gặp

| Triệu chứng | Cách xử lý |
|---|---|
| OOM khi train trên L4 | Bình thường — full SFT cần ~32,5 GB. Dùng Đường A hoặc LoRA (chương 5). |
| Hết ổ đĩa | Checkpoint trung gian 22–24 GB mỗi cái; giảm `save_total_limit` trong `chapter06/train_sft.py`. |
| `No module named 'chapterXX'` | Chạy lại ô cài đặt đầu tiên (nó `cd` vào `code/` và thêm vào `sys.path`). |
| Mất kết nối / runtime reset | Chạy lại từ ô cài đặt; lưu kết quả sang Drive sau mỗi bước dài. |

Tài liệu gốc của chương: [README.md](README.md).
