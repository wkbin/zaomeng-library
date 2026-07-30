# Zaomeng Library

本仓库是早梦小说蒸馏包的静态资源库。Web 和 Android 客户端请求根目录的 `index.json`，显示书目并下载对应的 `.zaomeng-run.zip` 导入。

当前公开索引地址：`https://raw.githubusercontent.com/wkbin/zaomeng-library/main/index.json`

## 目录与索引

```text
zaomeng-library/
|- index.json
`- books/
   |- hongloumeng.zaomeng-run.zip
   `- san-guo.zaomeng-run.zip
```

`index.json` 是唯一的书目接口，使用 UTF-8 JSON 和两空格缩进：

```json
{
  "version": 1,
  "updated_at": "2026-07-30T00:00:00Z",
  "books": [
    {
      "id": "hongloumeng",
      "title": "红楼梦",
      "created_by": "创建书卷的账号或署名",
      "summary": "书目简介",
      "version": "1.0.0",
      "release_notes": "该版本的内容说明",
      "download_url": "https://raw.githubusercontent.com/<account>/<repo>/main/books/hongloumeng.zaomeng-run.zip",
      "sha256": "小写 SHA-256 十六进制摘要",
      "size_bytes": 1234567
    }
  ]
}
```

## 客户端规则

1. 请求 `index.json`；无法解析、`version` 不受支持或 `books` 不是数组时，停止导入并提示更新客户端。
2. `id` 是稳定的书目主键，使用小写 ASCII 字母、数字和连字符，且在 `books` 内唯一；`title` 只用于展示。
3. 直接请求 `download_url` 下载包。当前图书馆使用 `raw.githubusercontent.com` 的完整 URL；客户端不应依赖 GitHub Releases。
4. 下载到临时文件后，校验字节数及 SHA-256；不匹配时删除临时文件，不得导入。
5. 校验成功后交由导入器读取 ZIP 根目录的 `package_manifest.json`，确认它是客户端支持的早梦包格式。客户端以 `id + version` 记录已导入条目；版本变化时提示用户重新导入。

## 入库规则

1. 原始包直接放入 `books/`，文件名为 `<book-id>.zaomeng-run.zip`。不得解压、重打包或修改包内文件。
2. 新增或替换包后，计算实际文件的 `size_bytes` 和 SHA-256，并同步更新对应的 `created_by`、`version`、`release_notes`、`download_url` 与 `updated_at`。`created_by` 表示创建蒸馏书卷的账号或署名，不是小说原作者。`release_notes` 只描述本次版本的新增、修订或已知限制。
3. 同一 `id` 的 `version` 更新意味着包内容变更。保留在仓库历史中可回溯；若要让旧版本长期可下载，应改用带版本的文件名或 GitHub Releases，避免覆盖 `books/` 下的旧文件。
4. 入库前确认 ZIP 可以打开，且根目录存在 `package_manifest.json`。包中的 `kind` 和 `schema_version` 必须受客户端支持。
5. 静态服务器应通过 HTTPS 提供 `GET` 和 `HEAD`，并允许跨域读取 `index.json` 和 ZIP：`Access-Control-Allow-Origin: *`。ZIP 必须保留原始字节和 `Content-Length`；支持 Range 请求可用于断点续传。

## 书卷入库申请

外部贡献者通过 GitHub 的“书卷包入库申请”Issue 表单提交书卷信息，并将完整的 `.zaomeng-run.zip` 文件直接拖入 Issue 正文。Issue 是审核入口，不会自动发布书卷；若无法附加文件，可填写备用下载地址。维护者必须完成以下检查后才将包放入 `books/` 并更新 `index.json`：

1. 下载的包可打开，且 ZIP 根目录存在受支持的 `package_manifest.json`。
2. 包的 `kind`、`schema_version` 和清单信息与申请内容兼容。
3. 实际文件大小和 SHA-256 已重新计算；申请中提供的摘要仅作交叉核对。
4. `book_id` 未被占用，或该申请是同一书卷的更高版本；创建者与版本说明已写入索引。
5. 申请者已确认其有权公开提交该包及其中内容。
