---
name: filestore-client
description: Use this skill whenever the user needs to call, wrap, document, or generate client code for the Wiscom filestore HTTP APIs from psmp_common_filestore, especially WiscomDataControl.java endpoints such as /send/wiscom/json, /filestore/wiscom/json, /file/download, /database, storeImage/storeFile helpers, or filestoreUrl configuration. This skill should trigger for Java/Spring client encapsulation, upload image/file bytes, download-and-store remote files, or reading filestore service URLs from config.
---

# Filestore Client

Use this skill to generate or modify client-side code that calls the Wiscom filestore service implemented by:

`src/main/java/com/wiscom/filestore/control/WiscomDataControl.java`

The service is a Spring Boot HTTP service. In the scanned project, the default HTTP port is configured in `config/application.properties`:

```properties
server.port=9223
```

Business behavior is selected by `config/application-business.properties` with `company=local|ftp|database|haikang|haikanghpsp|huazhi|other`.

## Core Endpoint

Prefer wrapping this endpoint for normal client use:

```http
POST {filestoreUrl}/send/wiscom/json
POST {filestoreUrl}/filestore/wiscom/json
Content-Type: application/json
```

Both paths are mapped to the same controller method and accept a JSON body matching `ReciveBean`:

```json
{
  "pass_time": "yyyyMMddHHmmss",
  "content": "<byte[] serialized by the JSON library, commonly Base64>",
  "file_suffix": "jpg",
  "head_path": "koala,alarm",
  "bottom_path": ""
}
```

The response is a plain string path or URL. Empty string or `"error"` usually indicates storage failure. Some backends can return a relative path; some return a full HTTP URL depending on `*.full.url` settings.

### Required Fields

- `content`: file bytes. Do not pass null unless the caller explicitly wants the service behavior of returning `null.<suffix>` in some storage modes.
- `file_suffix`: extension without a leading dot for generated file names, such as `jpg`, `png`, `mp4`.
- `pass_time`: timestamp in `yyyyMMddHHmmss`. If the caller omits it, client wrappers should default to current time in that format.
- `head_path`: comma-separated logical directory path. For local/default storage, use at least two levels such as `koala,alarm`.
- `bottom_path`: optional comma-separated or backend-specific tail path. Use empty string or null only when the target backend accepts it.

## Configuration Pattern

When generating Java/Spring client code, make `filestoreUrl` configurable. Use one of these property names unless the host project already has a convention:

```properties
filestore.url=http://127.0.0.1:9223/filestore/wiscom/json
```

or split base URL and endpoint:

```properties
filestore.base-url=http://127.0.0.1:9223
filestore.store-path=/filestore/wiscom/json
```

Prefer injecting configuration with `@Value` or `@ConfigurationProperties`. Keep the actual URL out of business code.

## Java Wrapper Template

When the user asks to encapsulate image upload or provides code similar to `storeImage`, generate a small static helper or Spring component like this. Adapt the HTTP utility to the target project's existing helper first.

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.HashMap;
import java.util.Map;

public class FilestoreClient {
    private static final DateTimeFormatter PASS_TIME_FORMAT =
            DateTimeFormatter.ofPattern("yyyyMMddHHmmss");

    private final String filestoreUrl;

    public FilestoreClient(String filestoreUrl) {
        this.filestoreUrl = filestoreUrl;
    }

    public String storeImage(byte[] imgBytes, String passTime) throws Exception {
        return storeFile(imgBytes, "jpg", passTime, "koala,alarm", "");
    }

    public String storeFile(byte[] content,
                            String fileSuffix,
                            String passTime,
                            String headPath,
                            String bottomPath) throws Exception {
        Map<String, Object> param = new HashMap<>();
        param.put("file_suffix", fileSuffix);
        param.put("pass_time", passTime == null
                ? LocalDateTime.now().format(PASS_TIME_FORMAT)
                : passTime);
        param.put("content", content);
        param.put("head_path", headPath);
        param.put("bottom_path", bottomPath == null ? "" : bottomPath);
        return HttpClientPoolUtil.jsonPostFromPool(
                filestoreUrl,
                ObjectToJson.objectToJson(param)
        );
    }
}
```

For a Spring-managed wrapper:

```java
@Component
public class FilestoreClient {
    @Value("${filestore.url}")
    private String filestoreUrl;

    public String storeImage(byte[] imgBytes, String passTime) throws Exception {
        Map<String, Object> imageParam = new HashMap<>();
        imageParam.put("file_suffix", "jpg");
        imageParam.put("pass_time", passTime == null
                ? LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss"))
                : passTime);
        imageParam.put("content", imgBytes);
        imageParam.put("head_path", "koala,alarm");
        imageParam.put("bottom_path", "");
        return HttpClientPoolUtil.jsonPostFromPool(
                filestoreUrl,
                ObjectToJson.objectToJson(imageParam)
        );
    }
}
```

If the target project uses Jackson/RestTemplate/WebClient instead of `HttpClientPoolUtil`, preserve the same JSON field names exactly.

## Other WiscomDataControl Endpoints

Only expose these in generated client code when the user needs them:

| Method | Path | Purpose | Request | Response |
| --- | --- | --- | --- | --- |
| GET | `/` | Returns storage node status only when `company=other`; otherwise null | none | object/null |
| GET | `/getNewInsertTime` | Returns last insert time map as JSON string | none | string |
| GET | `/filter?source=...` | Triggers filter integration | query `source` | `"ok"` |
| GET | `/test` | Thrift test helper | none | string |
| POST | `/send/wiscom/json` | Store supplied bytes | `ReciveBean` JSON | stored path/URL string |
| POST | `/filestore/wiscom/json` | Same as `/send/wiscom/json` | `ReciveBean` JSON | stored path/URL string |
| GET | `/database?url=...` | Reads image bytes from database table and writes image/png to response | query `url` | image stream |
| POST | `/file/download` | Downloads a remote file URL, stores it, and returns URL/path | `{"url":"...","file_suffix":"jpg"}` | string |

## `/file/download` Wrapper

Use this endpoint when the source is a remote URL and the filestore service should download and store it itself:

```json
{
  "url": "http://example.com/image.jpg",
  "file_suffix": "jpg"
}
```

The controller rejects null downloads and files smaller than 100 bytes by returning an empty string. It stores under hard-coded `head_path=video,illegal` and current `pass_time`.

## Backend-Specific Notes

- `local`: requires `pass_time` length 14 and at least two `head_path` levels. If `local.full.url=yes`, returns `http://{local.host.ip}:{local.http.port}/{local.http.pre.path}...`.
- `ftp`: `pass_time` may be null or empty for permanent storage, otherwise it must be length 14. `head_path` must have at least one level. If `file_suffix` contains `.`, the service treats it as a full file name.
- `database`: stores bytes in `database.write.table` and returns generated id. `/database?url={id}` reads from `database.read.table`.
- `haikang` / `haikanghpsp`: service delegates upload to Hik storage and can return full URL when `haikang*.full.url=yes`.
- `huazhi`: uploads through HuaZhi SDK using `huazhi.root.path.name`.
- default/normal path: requires `pass_time`, `file_suffix`, and `head_path`; `head_path` should have at least two comma-separated levels.

## Output Guidance

When responding to a user with generated code:

1. Include the config property they need to add, such as `filestore.url`.
2. Generate one focused client class or method rather than duplicating controller internals.
3. Preserve JSON field names: `file_suffix`, `pass_time`, `content`, `head_path`, `bottom_path`.
4. Default `pass_time` to `LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss"))`.
5. Keep `filestoreUrl` read from config; do not hard-code service IP/port in the method body.
6. Mention the default endpoint path as `/filestore/wiscom/json` unless the user asks for `/send/wiscom/json`.
7. Return the service response string directly unless the user asks to validate or normalize URLs.

