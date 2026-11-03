# Hướng dẫn chạy Chương 8: DPO and Advanced Alignment trên Google Colab

Notebook: [`chapter08_colab.ipynb`](chapter08_colab.ipynb)

## Mục tiêu

Căn chỉnh (align) model bằng DPO: chuẩn bị preference pairs, train LoRA-DPO trên model SFT, đánh giá/safety với model full DPO đã publish, và dùng Claude làm LLM-judge.

## Yêu cầu

| Hạng mục | Chi tiết |
|---|---|
| GPU | **L4** (24 GB) hoặc A100. Full-parameter DPO cần ≥ 80 GB (thường không có trên Colab) |
| Thời gian | ~1–1,5 giờ (đánh giá chiếm phần lớn); tải 2 model ~16 GB |
| API | Bonus LLM-judge dùng `ANTHROPIC_API_KEY` — vài trăm lời gọi, ước tính vài USD với Sonnet 5.5. |

## Cách mở notebook trên Google Colab

**Cách 1 – Mở trực tiếp từ GitHub (khuyến nghị):**
1. Push repo lên GitHub (repo `phuongtnd1994/ptnd-ModelAdaptationBook` phải ở chế độ *public*, hoặc bạn đăng nhập GitHub trong Colab để mở repo private).
2. Vào <https://colab.research.google.com> → **File › Open notebook › GitHub** → dán `phuongtnd1994/ptnd-ModelAdaptationBook` → chọn file `code/chapter08/chapter08_colab.ipynb`.
   Hoặc bấm badge **Open in Colab** ở đầu notebook.

**Cách 2 – Upload file:** **File › Upload notebook** → chọn `chapter08_colab.ipynb` từ máy.

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
| 0. Cài đặt | Cài package, tải `ch6-sft` và `ch8-dpo` từ Hugging Face. |
| 1. Test | `pytest chapter08/tests/`. |
| 2. Pairs | `prepare_preference_data` → `chapter08/data/preference_pairs/`. |
| 3. LoRA-DPO | `train_dpo --lora --model_dir /content/hub/ch6-sft` → `chapter08/runs/dpo_lora_run1/`. |
| 4. Eval | `eval_dpo` SFT vs DPO (Hub) trên test split. |
| 5. Safety | `safety_after_dpo`. |
| 6. Inference | Full DPO (Hub) và LoRA-DPO của bạn. |
| 7. Claude judge | Shim `/content/run_with_claude.py` + `eval_lora_dpo` → `dpo_lora_report_claude.json`. |
| 8. Full DPO | Chỉ chạy khi GPU ≥ 80 GB. |

## Kết quả mong đợi

DPO giữ hoặc cải thiện chất lượng so với SFT, không tụt safety. Report judge có điểm 1–5 theo tiêu chí và tỉ lệ thắng từng cặp (vị trí A/B được xáo ngẫu nhiên).

## Lỗi thường gặp

| Triệu chứng | Cách xử lý |
|---|---|
| `CUDA out of memory` khi train | Giữ `--lora`; giảm `--max_length 384`. |
| `RateLimitError` từ Claude | Giảm `--workers` xuống 2. |
| Không tìm thấy SFT model | Chạy lại ô tải từ Hugging Face. |
| `No module named 'chapterXX'` | Chạy lại ô cài đặt đầu tiên (nó `cd` vào `code/` và thêm vào `sys.path`). |
| pip báo `dependency conflicts` với `gradio`, `diffusers` | Bỏ qua được — các gói Colab cài sẵn mà sách không dùng. Miễn là ô in ra `Đã cài xong.` |
| Mất kết nối / runtime reset | Chạy lại từ ô cài đặt; lưu kết quả sang Drive sau mỗi bước dài. |

## Hiểu kiến thức của chương

Mở [`DPO_GIAI_THICH.html`](DPO_GIAI_THICH.html) bằng trình duyệt (tải file về rồi nhấp đúp) để đọc phần giải thích trực quan: ý tưởng, sơ đồ, giải thích code và cách đọc kết quả của chương.

Tài liệu gốc của chương: [README.md](README.md).
