# Auto Sync Forks

GitHub Actions workflow tự động đồng bộ tất cả forked repositories với upstream source.

## Tính năng

- Tự động chạy hàng ngày lúc 2:00 AM UTC
- Hỗ trợ chạy thủ công qua workflow_dispatch
- Sử dụng GraphQL API với fallback sang REST API khi lỗi
- Phân trang để xử lý số lượng lớn repositories
- Retry logic với exponential backoff
- Báo cáo chi tiết kết quả sync

## Cài đặt

### 1. Tạo Personal Access Token (PAT)

Vào GitHub Settings > Developer settings > Personal access tokens > Tokens (classic)

Cấp quyền:
- `repo` (Full control of private repositories)
- `workflow` (Update GitHub Action workflows)

### 2. Thêm Secret vào Repository

Settings > Secrets and variables > Actions > New repository secret

- Name: `PAT_TOKEN`
- Value: Token vừa tạo

### 3. Cấu hình Owner

Sửa biến `OWNER` trong file `sync-all-forks.yml`:

```yaml
OWNER: your-github-username
```

### 4. Tạo Workflow

Copy file `sync-all-forks.yml` vào `.github/workflows/` của repository.

## Cách hoạt động

1. Lấy danh sách tất cả forked repos qua GraphQL API
2. Nếu GraphQL fail → fallback sang REST API
3. Với mỗi fork:
   - Clone shallow (depth 1)
   - Fetch upstream default branch
   - Reset hard về upstream
   - Force push về origin
4. Báo cáo tổng kết: success/failed/skipped

## Tùy chỉnh

### Loại trừ repositories

Thêm tên repo vào biến `EXCLUDE`:

```bash
EXCLUDE="repo1 repo2 repo3"
```

### Thay đổi lịch chạy

Sửa cron expression:

```yaml
schedule:
  - cron: '0 2 * * *'  # Mỗi ngày lúc 2:00 AM UTC
```

## Lưu ý

- Workflow sẽ **force push** - các thay đổi local trên fork sẽ bị ghi đè
- Timeout: 60 phút
- Safety limit: tối đa 30 pages (~900 repos)
- Nếu push fail do workflow files, sẽ tự động giữ lại workflow files của fork

## License

MIT
