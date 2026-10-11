# Bài 4: Khai báo GitHub Secrets và cấu hình Đăng nhập Container Registry

## 1. Mục tiêu
- Tạo tệp mã hóa an toàn (Repository Secret) lưu trữ Token cá nhân trên GitHub.
- Cấu hình bước xác thực đăng nhập tự động vào GitHub Container Registry (GHCR).

## 2. Sản phẩm nộp bài
| File | Mô tả |
|------|-------|
| `.github/workflows/ci.yml` | Kịch bản CI có bước `docker/login-action@v3` dùng `secrets.REGISTRY_TOKEN` |
| Ảnh chụp `Settings -> Secrets and variables -> Actions` | Hiển thị biến `REGISTRY_TOKEN` đã khởi tạo (che giá trị token) |
| Ảnh chụp log tab `Actions` | Bước `Đăng nhập vào GitHub Container Registry` hiển thị `Login Succeeded` |

> Thư mục này (`bai4/`) là template. Khi nộp bài thật, copy file
> `bai4/.github/workflows/ci.yml` thành `.github/workflows/ci.yml` ở **root của repository**.

## 3. Bước 1 — Tạo Personal Access Token (classic)

1. Vào GitHub → avatar → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**.
2. **Generate new token (classic)**:
   - Note: `REGISTRY_TOKEN_BAI4`
   - Expiration: `90 days` (hoặc theo quy định lớp).
   - Tick quyền:
     - `write:packages` (bắt buộc — push image lên GHCR)
     - `read:packages` (tự tick kèm theo)
     - `delete:packages` (nên tick để dọn image test)
3. **Generate token** → **Copy ngay** (chỉ hiện 1 lần, dạng `ghp_...`).

## 4. Bước 2 — Tạo Repository Secret `REGISTRY_TOKEN`

1. Vào repository → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**.
2. Name: `REGISTRY_TOKEN`
3. Secret: dán token `ghp_...` vừa copy.
4. **Add secret** → chụp màn hình (thấy dòng `REGISTRY_TOKEN` + `Updated ...`) để nộp bài.

## 5. Bước 3 — File `ci.yml` (đã có sẵn trong thư mục này)

Điểm quan trọng đúng theo gợi ý đề bài:

```yaml
      - name: Đăng nhập vào GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}
```

Giải thích:
- `registry: ghcr.io` — đăng nhập GHCR, không phải DockerHub.
- `username: ${{ github.actor }}` — user đang trigger workflow.
- `password: ${{ secrets.REGISTRY_TOKEN }}` — lấy từ Repository Secret, **không bao giờ hardcode token**.

## 6. Kiểm tra

```bash
# copy template ra root repo rồi push
cp bai4/.github/workflows/ci.yml .github/workflows/ci.yml
git add .github/workflows/ci.yml
git commit -m "bai4: them login GHCR"
git push origin main
```

1. Mở tab **Actions** → chọn run mới nhất → job `Test Gradle & Login GHCR`.
2. Mở step **Đăng nhập vào GitHub Container Registry** → phải thấy:
   ```
   Login Succeeded
   ```
3. Chụp màn hình log này nộp kèm.

## 7. Lỗi thường gặp
| Lỗi | Nguyên nhân → Cách sửa |
|-----|------------------------|
| `ERROR: failed to authorize: ... 401 Unauthorized` | Token sai / hết hạn / thiếu `write:packages` → tạo lại PAT classic, update Secret |
| `The input 'password' is empty` / `username blank` | Chưa tạo Secret `REGISTRY_TOKEN` hoặc sai tên (phân biệt hoa thường) → kiểm tra Settings → Secrets |
| Dùng PAT fine-grained mà vẫn 401 | PAT fine-grained phải mở quyền `Packages: read+write` cho đúng repo → nên dùng **classic** cho đơn giản theo đề |
| Login thành công nhưng push vẫn denied | Package `ghcr.io/<owner>/<repo>` chưa public / chưa cấp quyền `GITHUB_TOKEN` → vào package settings cấp quyền, hoặc dùng PAT có `write:packages` |
