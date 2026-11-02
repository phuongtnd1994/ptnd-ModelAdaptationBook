# Hướng dẫn chạy Chương 9: Managing Model Evolution, Drift & Versioning trên Google Colab

Notebook: [`chapter09_colab.ipynb`](chapter09_colab.ipynb)

## Mục tiêu

Thực hành vận hành model: registry (đăng ký/promote/rollback), phát hiện drift, canary prompts, rollback demo, safety monitor; bonus Claude viết báo cáo vận hành.

## Yêu cầu

| Hạng mục | Chi tiết |
|---|---|
| GPU | CPU cho Stage 1, 2, 4; **L4** (hoặc T4) cho Stage 3 và 5 |
| Thời gian | CPU: vài phút · GPU stages: ~20–30 phút; tải 2 model ~16 GB |
| API | Bonus dùng `ANTHROPIC_API_KEY` (1 lời gọi). |

## Cách mở notebook trên Google Colab

**Cách 1 – Mở trực tiếp từ GitHub (khuyến nghị):**
1. Push repo lên GitHub (repo `phuongtnd1994/ptnd-ModelAdaptationBook` phải ở chế độ *public*, hoặc bạn đăng nhập GitHub trong Colab để mở repo private).
2. Vào <https://colab.research.google.com> → **File › Open notebook › GitHub** → dán `phuongtnd1994/ptnd-ModelAdaptationBook` → chọn file `code/chapter09/chapter09_colab.ipynb`.
   Hoặc bấm badge **Open in Colab** ở đầu notebook.

**Cách 2 – Upload file:** **File › Upload notebook** → chọn `chapter09_colab.ipynb` từ máy.

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
| 0. Cài đặt | Cài package, tải `ch6-sft` và `ch8-dpo`. |
| 1. Test | `pytest chapter09/tests/`. |
| 2. Registry | Registry riêng ở `chapter09/runs/colab_registry`: register v1 (SFT) → promote → register v2 (DPO) → promote → rollback. |
| 3. Drift | Train vs valid (ít drift) và train vs shifted production (drift cao). |
| 4. Canary | Baseline SFT, so DPO với baseline. |
| 5. Rollback demo | `rollback_demo` với registry demo riêng. |
| 6. Safety | Safety monitor SFT, rồi DPO so với baseline. |
| 7. Bonus Claude | Tổng hợp report → báo cáo tiếng Việt. |

## Kết quả mong đợi

Drift score thấp cho valid, cao cho shifted set; rollback đưa v1 về production; safety pass rate có thể quanh ~56% (xem README chương 9 để hiểu vì sao).

## Lỗi thường gặp

| Triệu chứng | Cách xử lý |
|---|---|
| `Version tag already exists` | Xoá `chapter09/runs/colab_registry` rồi chạy lại Stage 1. |
| OOM ở canary/safety | Cần ≥ 8 GB VRAM; vẫn chạy được Stage 1, 2, 4 trên CPU. |
| `No module named 'chapterXX'` | Chạy lại ô cài đặt đầu tiên (nó `cd` vào `code/` và thêm vào `sys.path`). |
| pip báo `dependency conflicts` với `gradio`, `diffusers` | Bỏ qua được — các gói Colab cài sẵn mà sách không dùng. Miễn là ô in ra `Đã cài xong.` |
| Mất kết nối / runtime reset | Chạy lại từ ô cài đặt; lưu kết quả sang Drive sau mỗi bước dài. |

Tài liệu gốc của chương: [README.md](README.md).
