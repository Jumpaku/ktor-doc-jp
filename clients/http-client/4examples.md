---
title: サンプル
category: clients
permalink: /clients/http-client/examples.html
caption: HTTPクライアントのサンプル
ktor_version_review: 1.2.0
---

## Ktorサーバ・クライアント間でのJSONの相互変換

{: #example-json }

```kotlin
fun main(args: Array<String>) {
    val server = embeddedServer(
        Netty,
        port = 8080,
        module = Application::mymodule
    ).apply {
        start(wait = false)
    }

    runBlocking {
        val client = HttpClient(Apache) {
            install(JsonFeature) {
                serializer = GsonSerializer {
                    // .GsonBuilder
                    serializeNulls()
                    disableHtmlEscaping()
                }
            }
        }

        val message = client.post<HelloWorld> {
            url("http://127.0.0.1:8080/")
            contentType(ContentType.Application.Json)
            body = HelloWorld(hello = "world")
        }

        println("CLIENT: Message from the server: $message")

        client.close()
        server.stop(1L, 1L, TimeUnit.SECONDS)
    }
}

data class HelloWorld(val hello: String)

fun Application.mymodule() {
    install(ContentNegotiation) {
        gson {
            setPrettyPrinting()
        }
    }
    routing {
        post("/") {
            val message = call.receive<HelloWorld>()
            println("SERVER: Message from the client: $message")
            call.respond(HelloWorld(hello = "response"))
        }
    }
}
```

[ktor-samples](https://github.com/ktorio/ktor-samples)や[ktor-exercises](https://github.com/ktorio/ktor-exercises)リポジトリにサンプル・exerciseがあります。
{: .note }
