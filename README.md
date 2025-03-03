
[[中文]](https://github.com/vacuityv/ai-java-sdk/tree/develop) [[English]](https://github.com/vacuityv/ai-java-sdk/blob/develop/README-eng.md)

# AI-Java-Sdk

> 欢迎体验个人对接的各大厂商AI的交互网站：[AI-CHAT 交互网站](https://chat.vacuity.me/)

为使用各大AI厂商提供的API创建的java sdk，目前支持Google Gemini 和 Claude AI 以及 openai 的 chat、file 和 assistant (v2) 部分功能。


## 支持的 Claude api
- [对话 (含vision，支持function)](https://docs.anthropic.com/claude/reference/messages_post)
- [流式对话 (含vision)](https://docs.anthropic.com/claude/reference/messages-streaming)

## 支持的 Google Gemini
- [对话 (含vision)](https://ai.google.dev/tutorials/rest_quickstart)
- [流式对话 (含vision)](https://ai.google.dev/tutorials/rest_quickstart)

## 支持的 openAI

> 支持函数调用，请参考 OpenaiTest（函数部分实现大量参考了 https://github.com/TheoKanning/openai-java ）

- [对话 (含vision)](https://platform.openai.com/docs/api-reference/chat/create)
- [流式对话 (含vision)](https://platform.openai.com/docs/api-reference/chat/streaming)
- [文件](https://platform.openai.com/docs/api-reference/files)
- [Assistant (含stream)](https://platform.openai.com/docs/api-reference/assistants)
- [Image](https://platform.openai.com/docs/api-reference/images)



## Importing

### Maven
```xml
<dependency>
    <groupId>me.vacuity.ai.sdk</groupId>
    <artifactId>ai-java-sdk</artifactId>
    <version>${version}</version>       
</dependency>
```
在这里可以查看最新的版本号：[Maven Central](https://central.sonatype.com/artifact/me.vacuity.ai.sdk/ai-java-sdk)

## 使用

普通对话：

```java
@Test
public void chat() {
    ClaudeClient client = new ClaudeClient(API_KEY);
    List<ChatMessage> messages = new ArrayList<>();
    messages.add(new ChatMessage("user", "introduce yourself pls"));
    ChatRequest request = ChatRequest.builder()
            .model("claude-3-opus-20240229")
            .messages(messages)
            .maxTokens(1024)
            .build();
    try {
        ChatResponse response = client.chat(request);
        System.out.println(response);
    } catch (VacException e) {
        if (e.getDetail() != null) {
            System.out.println(e.getDetail().getError().getMessage());
        }
    }
}
```

流式对话：

```java
@Test
public void streamChat() {
    ClaudeClient client = new ClaudeClient(API_KEY);
    List<ChatMessage> messages = new ArrayList<>();
    messages.add(new ChatMessage("user", "鲁迅为什么打周树人"));
    ChatRequest request = ChatRequest.builder()
            .model("claude-3-opus-20240229")
            .messages(messages)
            .maxTokens(1024)
            .build();
    Flowable<StreamChatResponse> response = client.streamChat(request);
    response.doOnNext(s -> {
        if ("content_block_delta".equals(s.getType())) {
            ChatMessageContent content = s.getDelta();
            System.out.print(content.getText());
        } else if ("error".equals(s.getType())) {
            System.out.println(s.getError().getMessage());
        }
    }).blockingSubscribe();
}
```

openAI vision：

```java
@Test
public void vision() throws IOException {
    String imagePath = "222.jpg";
    Path path = Paths.get(imagePath);
    // read file
    byte[] imageBytes = Files.readAllBytes(path);

    InputStream is = new BufferedInputStream(new FileInputStream(imagePath));
    String mimeType = URLConnection.guessContentTypeFromStream(is);

    // convert image to base64 data
    String base64Image = Base64.getEncoder().encodeToString(imageBytes);
    base64Image = "data:" + mimeType + ";base64," + Base64.getEncoder().encodeToString(imageBytes);
    String url = "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg";

    OpenaiClient client = new OpenaiClient(API_KEY, Duration.ofSeconds(120));
    List<ChatMessage> messages = new ArrayList<>();

    ChatMessageContent content = new ChatMessageContent();
    ChatMessageContent.ImageUrl imageUrl = new ChatMessageContent.ImageUrl();
    // imageUrl.setUrl(url);
    imageUrl.setUrl(base64Image);
    content.setType("image_url");
    content.setImageUrl(imageUrl);
    ChatMessageContent content2 = new ChatMessageContent();
    content2.setType("text");
    content2.setText("what is this?");

    ChatMessage chatMessage = new ChatMessage("user", Arrays.asList(content, content2));
    messages.add(chatMessage);

    ChatRequest request = ChatRequest.builder()
            .model("gpt-4-vision-preview")
            .messages(messages)
            .build();
    Flowable<StreamChatResponse> response = client.streamChat(request);
    response.doOnNext(s -> {
        System.out.println(s.getSingleContent());
    }).blockingSubscribe();
}
```


### 自定义地址和超时时间

```java
ClaudeClient client = new ClaudeClient(API_KEY, Duration.ofSeconds(100), "https://example.com");
```


### 使用http代理

```java
@Test
@Test
public void proxyChat() {
    String host = "127.0.0.1";
    int port = 7890;
    Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress(host, port));

    ClaudeClient client = new ClaudeClient(API_KEY, Duration.ofSeconds(60), proxy);

    List<ChatMessage> messages = new ArrayList<>();
    messages.add(new ChatMessage("user", "introduce yourself pls"));
    ChatRequest request = ChatRequest.builder()
            .model("claude-3-opus-20240229")
            .messages(messages)
            .maxTokens(1024)
            .build();
    try {
        ChatResponse response = client.chat(request);
        System.out.println(response.getContent().get(0).getText());
    } catch (VacSdkException e) {
        if (e.getDetail() != null) {
            System.out.println(e.getDetail().getError().getMessage());
        }
    }
}
```
## 其他

你可以在 CludeTest 和 GeminiTest 以及 OpenaiTest/OpenaiAssistantTest 查看代码示例

## FAQ
### 支持什么模型
目前支持Claude ai 和 Google Gemini 以及 Openai 的部分模型

### openai 还有很多其他功能，这个sdk不支持吗
openai 目前在github上都有对应的sdk支持（比如：https://github.com/TheoKanning/openai-java ），所以不是紧急的需求，也许会在未来支持

## License
Published under the MIT License

## 如果您有余力，欢迎贡献代码 or Money

<img width="200" height="200" src="https://github.com/vacuityv/self-pay/blob/main/vac-wechat.jpg"/>


<img width="200" height="200" src="https://github.com/vacuityv/self-pay/blob/main/vac-alipay.jpg"/>
