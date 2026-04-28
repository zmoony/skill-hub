```text
/skill-creator
扫描D:\project\code\wiscom\2025\common\psmp_common_filestore，将D:\project\code\wiscom\2025\common\psmp_common_filestore\src\main\java\com\wiscom\filestore\control\WiscomDataControl.java里的接口封装成skill

## 示例
public static String storeImage(byte[] imgBytes, String passTime,String filestoreUrl) throws Exception {  
    Map<String, Object> imageParam = new HashMap<>();  
    imageParam.put("file_suffix", "jpg");  
    imageParam.put("pass_time", passTime==null? LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss")):passTime);  
    imageParam.put("content", imgBytes);  
    imageParam.put("head_path", "koala,alarm");  
    imageParam.put("bottom_path", "");  
    return HttpClientPoolUtil.jsonPostFromPool(filestoreUrl,  ObjectToJson.objectToJson(imageParam));  
}

## 可配置的
filestoreUrl 配置文件读取

## 输出
生成的skill落地到D:\文档\Obsidian\skill-hub\skills\wiscom，名字为filestore-client

```