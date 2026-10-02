# 書籍管理 REST API

以 **FastAPI** 實作的書籍資料 CRUD API，資料來源為博客來 AI／LLM 類書籍，搭配 SQLite 儲存。
重點放在 **RESTful 設計慣例**、**輸入驗證** 與 **正確的 HTTP 狀態碼**。

## API 端點

| 方法 | 路徑 | 說明 | 成功狀態碼 |
| --- | --- | --- | --- |
| `GET` | `/` | 服務健康檢查 | `200` |
| `GET` | `/books` | 取得書籍列表，支援 `skip` / `limit` 分頁 | `200` |
| `GET` | `/books/{book_id}` | 取得單一書籍，查無資料回 `404` | `200` |
| `POST` | `/books` | 新增書籍 | `201 Created` |
| `PUT` | `/books/{book_id}` | 更新書籍，僅更新有傳入的欄位 | `200` |
| `DELETE` | `/books/{book_id}` | 刪除書籍 | `204 No Content` |

## 設計重點

**分層架構** — 三個模組各司其職，路由層不直接碰 SQL：

```
main.py       # 路由與 HTTP 語意（狀態碼、404 處理）
models.py     # Pydantic 結構定義與驗證規則
database.py   # SQLite 存取層，統一管理連線開關
```

**輸入驗證**（Pydantic）

- `price` 以 `Field(..., gt=0)` 約束必須大於 0，不合法的請求由框架擋下並回 `422`，不會進到資料庫。
- `Create` 與 `Update` 分開定義：新增時必填欄位不可省略，更新時所有欄位皆為選填，實作部分更新（partial update）。
- `BookResponse` 獨立為回應模型，確保 `id`、`created_at` 等由資料庫產生的欄位出現在輸出。

**分頁參數驗證**

- `skip` 限制 `ge=0`、`limit` 限制 `gt=0, le=100`，避免一次撈取過量資料。

**資料庫存取**

- 所有查詢使用**參數化 SQL**（`?` 佔位），不以字串拼接組 SQL，避免注入風險。
- 連線以 `try / finally` 確保 cursor 與 connection 一定關閉。
- 設定 `row_factory = sqlite3.Row`，讓查詢結果可直接轉為 dict 回傳。

## 執行方式

```bash
pip install -r requirements.txt
uvicorn main:app --reload
```

啟動後開啟互動式文件：

- Swagger UI — http://127.0.0.1:8000/docs
- ReDoc — http://127.0.0.1:8000/redoc

## 執行結果

**啟動與自動產生的 API 文件**

| Swagger UI | ReDoc |
| --- | --- |
| ![Swagger](imgs/swagger.png) | ![ReDoc](imgs/redoc.png) |

**Postman 驗證**

查詢列表（含分頁）

![GET /books](imgs/postman_getBooks.png)

新增書籍（回傳 201 與建立後的資料）

![POST /books](imgs/postman_post.png)

刪除書籍（回傳 204）

![DELETE /books](imgs/postman_del.png)

---

<sub>本專案為課程實作，資料僅供學習用途。</sub>
