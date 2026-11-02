# Hướng dẫn chạy Chương 1: Vì sao cần Model Adaptation? trên Google Colab

Notebook: [`chapter01_colab.ipynb`](chapter01_colab.ipynb)

## Mục tiêu

Tái hiện sidebar §1.6: so sánh câu trả lời của model gốc, model LoRA (chương 5) và model SFT (chương 6) cho cùng một câu hỏi; bonus so sánh với Claude.

## Yêu cầu

| Hạng mục | Chi tiết |
|---|---|
| GPU | L4 hoặc A100 (T4 chạy được inference nhưng chậm) |
| Thời gian | ~10–15 phút (lần đầu tải model ~8 GB + adapter + model SFT ~8 GB) |
| API | Có dùng `ANTHROPIC_API_KEY` ở phần bonus (1 lời gọi, chi phí không đáng kể). |

## Cách mở notebook trên Google Colab

**Cách 1 – Mở trực tiếp từ GitHub (khuyến nghị):**
1. Push repo lên GitHub (repo `phuongtnd1994/ptnd-ModelAdaptationBook` phải ở chế độ *public*, hoặc bạn đăng nhập GitHub trong Colab để mở repo private).
2. Vào <https://colab.research.google.com> → **File › Open notebook › GitHub** → dán `phuongtnd1994/ptnd-ModelAdaptationBook` → chọn file `code/chapter01/chapter01_colab.ipynb`.
   Hoặc bấm badge **Open in Colab** ở đầu notebook.

**Cách 2 – Upload file:** **File › Upload notebook** → chọn `chapter01_colab.ipynb` từ máy.

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
| 0. Cài đặt | Clone repo, `pip install -e .[dev]`, kiểm tra GPU, nạp API key, tải `ch5-lora` và `ch6-sft` từ Hugging Face. |
| 1. Base-only | `python -m chapter01.run_sidebar_example --base_only` — chỉ chạy Qwen3-4B gốc. |
| 2. Ba cấu hình | Chạy lại với `--lora_dir /content/hub/ch5-lora --sft_dir /content/hub/ch6-sft`, in `sidebar_outputs.json`. |
| 3. Bonus Claude | Gửi cùng câu hỏi tới `claude-opus-5-5` để so sánh. |

## Kết quả mong đợi

Model gốc tự tin bịa ra thông tin chính sách và thêm câu rào đón cuối bài; model LoRA/SFT cũng bịa nhưng *bỏ* câu rào đón — đó là điểm sidebar muốn chỉ ra: fine-tune trên dữ liệu instruction chung không chữa được việc bịa (confabulation).

## Lỗi thường gặp

| Triệu chứng | Cách xử lý |
|---|---|
| `CUDA out of memory` | Dùng L4/A100, hoặc chỉ chạy bước 1 (base-only). |
| Tải model chậm | Thêm `HF_TOKEN` vào Secrets. |
| `No module named 'chapterXX'` | Chạy lại ô cài đặt đầu tiên (nó `cd` vào `code/` và thêm vào `sys.path`). |
| pip báo `dependency conflicts` với `gradio`, `diffusers` | Bỏ qua được — các gói Colab cài sẵn mà sách không dùng. Miễn là ô in ra `Đã cài xong.` |
| Mất kết nối / runtime reset | Chạy lại từ ô cài đặt; lưu kết quả sang Drive sau mỗi bước dài. |

Tài liệu gốc của chương: [README.md](README.md).
