+++
title = "在 Cloud Run 使用 Cloud Storage Volume 的一些注意事項"
date = 2026-04-13
draft = false
categories = ["GCP"]
+++

分享我最近在 GCP 上部署掛載 Cloud Storage Volume 的 Cloud Run Service時，遇到的一些值得一提的經驗（~坑~）。

##  Container 無故結束運行且未回報錯誤

通常在建立掛載 Cloud Storage Volume 的 Cloud Run Service 時，執行 Create Service 的 **Service Account** 應具備 **Storage Object User** 或 **Admin** Role。 如果權限不足，可能會見到以下狀況：

1. 在 Log Explorer 中看到與 Container Exit 相關的錯誤：

    ```text
    "textPayload": "Application exec likely failed"
    "textPayload": "terminated: Application failed to start: The container may have exited abnormally."
    ```

2. Log Explorer 顯示 Cloud Storage Volume 似乎已成功掛載：

    ```text
    "textPayload": "time="DD/mm/YYYY HH:MM:SS.ffffff" severity=INFO message="Mounting file system \"some-filesystem\"..." mount-id=some-filesystem-xxxxxxxx"
    ```

3. 無論如何調整 Entrypoint 或加入更多 Logs，Container 仍會如第 1 點所述直接結束，且不會出現預期的 Log。

與 Azure Container Apps 或 AWS ECS 不同，在 GCP Cloud Run 上建立掛載 Network Filesystem Volumes 的服務時，缺乏適當權限不會讓操作立即失敗。相反地，使用者仍會看到檔案系統像正常情況一般顯示掛載成功，但容器的 Entrypoint 卻無法正常執行。需要特別小心。

## Cloud Storage Volume 上的檔案存取問題

小心如下的錯誤訊息：

```text
"textPayload": "time="DD/mm/YYYY HH:MM:SS.ffffff" severity=ERROR message="BufferedWriteHandler.OutOfOrderError for object: db.sqlite-shm, expectedOffset: 0, actualOffset: 4095" mount-id=some-filesystem-xxxxxxxx"
"textPayload": "time="DD/mm/YYYY HH:MM:SS.ffffff" severity=INFO message="Out of order write detected. File db.sqlite-shm will now use legacy staged writes. Streaming writes is supported for sequential writes to new/empty files. For more details, see: https://github.com/GoogleCloudPlatform/gcsfuse/blob/master/docs/semantics.md#writes" mount-id=some-filesystem-xxxxxxxx"
```

在我遇到的情況中，我嘗試在 Cloud Storage Volume 上建立 SQLite 資料庫，並使用 [better-sqlite3](https://github.com/WiseLibs/better-sqlite3) 進行存取。 結果發現資料僅被寫入 Container 的記憶體的副本中，而沒有真正被寫入 Cloud Storage 上的資料庫檔案，同時在Log Explorer中觀察到了上述 INFO Level 的錯誤訊息。 我也試過確認錯誤訊息中的[連結](https://github.com/GoogleCloudPlatform/gcsfuse/blob/master/docs/semantics.md#writes)，但並未找到能解釋此情況的資訊。 雖然在 [Reddit 上](https://www.reddit.com/r/googlecloud/comments/1afm74h/sqlite_running_in_cloud_run_with_gcs_volume_mount/)有人成功實現了這個架構（雖然延遲很高），但發文者使用的是 Python Flask 和 SQLite3，運作方式可能與 Node.js 不同。

我的假設是，Object Store Services 對於 Partial Write 或 Streaming 相關的操作可能存在限制。Object Storage Service 的設計初衷通常是針對**檔案本身**進行操作，而非針對其**內容**。在其他[討論](https://www.reddit.com/r/aws/comments/dplfoa/why_is_s3fs_such_a_bad_idea/)中，也有人探討過將 Object Storage Services 當作 Filesystem 所可能產生的問題。

針對這個情況，我個人的最佳建議是：除非能確保對 Cloud Storage Volume 的檔案操作皆為檔案層級的操作，否則使用 EFS 會更保險。 以我為例，最終我還是將專案改寫為使用 PostgreSQL 並搭配 Cloud SQL 作為資料庫。
