# Hướng dẫn chạy Chương 4: In-Context Learning, Few-Shot, and RAG trên Google Colab

Notebook: [`chapter04_colab.ipynb`](chapter04_colab.ipynb)

## Mục tiêu

Chạy các kỹ thuật thay đổi *input* (không train): few-shot, many-shot, prompt validator, RAG tối giản và đánh giá RAG; bonus dùng Claude làm generator cho RAG.

## Yêu cầu

| Hạng mục | Chi tiết |
|---|---|
| GPU | CPU đủ cho backend `mock`/`hash`/`st`; L4/A100 cho backend `hf` (Qwen3-4B) |
| Thời gian | CPU: vài phút · backend `hf`: thêm ~10–20 phút |
| API | Bonus RAG dùng `ANTHROPIC_API_KEY` (3 lời gọi). |

## Cách mở notebook trên Google Colab

**Cách 1 – Mở trực tiếp từ GitHub (khuyến nghị):**
1. Push repo lên GitHub (repo `phuongtnd1994/ptnd-ModelAdaptationBook` phải ở chế độ *public*, hoặc bạn đăng nhập GitHub trong Colab để mở repo private).
2. Vào <https://colab.research.google.com> → **File › Open notebook › GitHub** → dán `phuongtnd1994/ptnd-ModelAdaptationBook` → chọn file `code/chapter04/chapter04_colab.ipynb`.
   Hoặc bấm badge **Open in Colab** ở đầu notebook.

**Cách 2 – Upload file:** **File › Upload notebook** → chọn `chapter04_colab.ipynb` từ máy.

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
| 1. Test | `pytest chapter04/tests/` |
| 2. Few-shot | Backend `mock` (CPU) rồi `hf` (GPU), in accuracy top-1. |
| 3. Many-shot | Chọn k ví dụ giống câu hỏi nhất (sentence-transformers hoặc hash). |
| 4. Validator | Chạy prompt nhiều lần, đo accuracy trung bình và tỉ lệ bất đồng. |
| 5. RAG | `retrieve` và `answer` (LLM giả). |
| 6. Đánh giá RAG | Precision@k, Hit@k, Hit@1 → `chapter04/runs/rag_eval.json`. |
| 7. Bonus Claude | Câu trả lời có căn cứ từ tài liệu; câu không có trong tài liệu phải được trả lời "không biết". |

## Kết quả mong đợi

Test pass; backend `hf` cho accuracy cao hơn hẳn `mock`; Hit@3 của RAG cao; Claude từ chối trả lời câu hỏi không có trong context.

## Lỗi thường gặp

| Triệu chứng | Cách xử lý |
|---|---|
| Backend `hf` OOM | Thêm `--model meta-llama/Llama-3.2-3B-Instruct` hoặc dùng backend `mock`. |
| `sentence-transformers` lỗi | Dùng `--backend hash`. |
| `No module named 'chapterXX'` | Chạy lại ô cài đặt đầu tiên (nó `cd` vào `code/` và thêm vào `sys.path`). |
| Mất kết nối / runtime reset | Chạy lại từ ô cài đặt; lưu kết quả sang Drive sau mỗi bước dài. |

Tài liệu gốc của chương: [README.md](README.md).
