# Hướng dẫn chạy Chương 3: What Data Do I Need for Model Adaptation? trên Google Colab

Notebook: [`chapter03_colab.ipynb`](chapter03_colab.ipynb)

## Mục tiêu

Chạy 3 phần của chương 3: DatasetManifest, pipeline sinh dữ liệu tổng hợp 6 bước dùng Claude làm teacher/judge, và thí nghiệm chất lượng dữ liệu (4 lần train LoRA).

## Yêu cầu

| Hạng mục | Chi tiết |
|---|---|
| GPU | CPU đủ cho phần 1–2; **L4 hoặc A100** cho phần 3 (thí nghiệm dùng `bf16=True`) |
| Thời gian | Phần 1: vài giây · Phần 2: 5–10 phút · Phần 3: 25–35 phút |
| API | **Bắt buộc** `ANTHROPIC_API_KEY` cho phần 2. Với `N_PER_CAT = 15` và `claude-sonnet-5-5` chi phí khoảng vài cent đến < $1. |

## Cách mở notebook trên Google Colab

**Cách 1 – Mở trực tiếp từ GitHub (khuyến nghị):**
1. Push repo lên GitHub (repo `phuongtnd1994/ptnd-ModelAdaptationBook` phải ở chế độ *public*, hoặc bạn đăng nhập GitHub trong Colab để mở repo private).
2. Vào <https://colab.research.google.com> → **File › Open notebook › GitHub** → dán `phuongtnd1994/ptnd-ModelAdaptationBook` → chọn file `code/chapter03/chapter03_colab.ipynb`.
   Hoặc bấm badge **Open in Colab** ở đầu notebook.

**Cách 2 – Upload file:** **File › Upload notebook** → chọn `chapter03_colab.ipynb` từ máy.

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
| 0. Cài đặt | Cài `.[dev,chapter03]` (thêm anthropic, matplotlib, scikit-learn, bitsandbytes). |
| 1. Manifest | `python -m chapter03.ch03_datasetmanifest` — tạo manifest, diff, kiểm tra retention. |
| 2. Synthetic | Thay `call_teacher` bằng hàm gọi Claude, rồi chạy lần lượt: load seeds → build prompt → generate → quality gate → distribution check → mix & save vào `ch03_synthetic_output/`. |
| 3. Thí nghiệm | `python -m chapter03.ch03_data_quality_explore` → `ch03_quality_experiment/results.json` + `accuracy_chart.png`. |
| 4. Lưu Drive | Tuỳ chọn. |

## Kết quả mong đợi

Phần 2: 45 ứng viên (15×3 nhãn), phần lớn qua quality gate, manifest có SHA-256. Phần 3: A/B/C ~0.96–0.98 accuracy, D (20% nhãn bị lật) tụt còn ~0.84 — nhãn mâu thuẫn hại model hơn nhãn "mờ".

## Lỗi thường gặp

| Triệu chứng | Cách xử lý |
|---|---|
| `AuthenticationError` | Kiểm tra lại `ANTHROPIC_API_KEY` trong Secrets và bật Notebook access. |
| Không ví dụ nào qua quality gate | Tăng `N_PER_CAT` hoặc đổi `TEACHER_MODEL = "claude-opus-5-5"`. |
| `Can't pickle ... safe_open` | `pip install "safetensors<0.6"` rồi Runtime > Restart session. |
| OOM ở phần 3 | Đặt `LOAD_IN_4BIT = True` trong `chapter03/ch03_data_quality_helpers.py`. |
| `No module named 'chapterXX'` | Chạy lại ô cài đặt đầu tiên (nó `cd` vào `code/` và thêm vào `sys.path`). |
| Mất kết nối / runtime reset | Chạy lại từ ô cài đặt; lưu kết quả sang Drive sau mỗi bước dài. |

Tài liệu gốc của chương: [README.md](README.md).
