---
title: CallId
caption: CallId
category: servers
permalink: /servers/features/call-id.html
feature:
  artifact: io.ktor:ktor-server-core:$ktor_version
  class: io.ktor.features.CallId
ktor_version_review: 1.0.0
---

CallId Featureを使うと、リクエスト/コールを識別ができるようになり、
[CallLogging](/servers/features/call-logging.html) Featureと一緒に動作させることができます。

## Call IDの生成 

```kotlin
install(CallId) {
    // Tries to retrieve a callId from an ApplicationCall. You can add several retrievers and all will be executed coalescing until one of them is not null.  
    retrieve { // call: ApplicationCall ->
        call.request.header(HttpHeaders.XRequestId)
    }
    
    // If can't retrieve a callId from the ApplicationCall, it will try the generate blocks coalescing until one of them is not null.
    val counter = atomic(0)
    generate { "generated-call-id-${counter.getAndIncrement()}" }
    
    // Once a callId is generated, this optional function is called to verify if the retrieved or generated callId String is valid. 
    verify { callId: String ->
        callId.isNotEmpty()
    }
    
    // Allows to process the call to modify headers or generate a request from the callId
    reply { call: ApplicationCall, callId: String ->
    
    }

    // Retrieve the callId from a headerName
    retrieveFromHeader(headerName: String)
    
    // Automatically updates the response with the callId in the specified headerName
    replyToHeader(headerName: String)
    
    // Combines both: retrieveFromHeader and replyToHeader in one single call
    header(headerName: String)
}
```

## [CallLogging](/servers/features/call-logging.html)の拡張
{: #call-logging-interop }

CallId Featureは`callIdMdc`拡張関数を含んでおり、この関数はCallLoggingの設定を行う際に利用されます。
この関数は`callId`を指定したキーに紐付けMDCコンテキストにputします。

```kotlin
install(CallLogging) {
    callIdMdc("mdc-call-id")
}
```
