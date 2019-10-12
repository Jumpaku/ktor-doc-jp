---
title: アップロード
caption: HTTPアップロードのハンドリング
category: servers
keywords: multipart receiving reading files
permalink: /servers/uploads.html
---

KtorはHTTPによるアップロードをサポートします。
[他の種類のコンテンツを受け取る場合](/servers/calls/requests.html)同様に動作します。

[Youkubeサンプル](/samples/app/youkube.html)を見ればこの機能についてのフルのサンプルが確認できます。
{: .note.example }

## multipartを使ったファイル受信

```kotlin
val multipart = call.receiveMultipart()
multipart.forEachPart { part ->
    when (part) {
        is PartData.FormItem -> {
            if (part.name == "title") {
                title = part.value
            }
        }
        is PartData.FileItem -> {
            val ext = File(part.originalFileName).extension
            val file = File(uploadDir, "upload-${System.currentTimeMillis()}-${session.userId.hashCode()}-${title.hashCode()}.$ext")
            part.streamProvider().use { input -> file.outputStream().buffered().use { output -> input.copyToSuspend(output) } }
            videoFile = file
        }
    }

    part.dispose()
}


suspend fun InputStream.copyToSuspend(
    out: OutputStream,
    bufferSize: Int = DEFAULT_BUFFER_SIZE,
    yieldSize: Int = 4 * 1024 * 1024,
    dispatcher: CoroutineDispatcher = Dispatchers.IO
): Long {
    return withContext(dispatcher) {
        val buffer = ByteArray(bufferSize)
        var bytesCopied = 0L
        var bytesAfterYield = 0L
        while (true) {
            val bytes = read(buffer).takeIf { it >= 0 } ?: break
            out.write(buffer, 0, bytes)
            if (bytesAfterYield >= yieldSize) {
                yield()
                bytesAfterYield %= yieldSize
            }
            bytesCopied += bytes
            bytesAfterYield += bytes
        }
        return@withContext bytesCopied
    }
}
```
