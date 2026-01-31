# LangChain4j初级开发
多余的话不说了，IDEA 的下载和项目的创建都是必须的。<br/>

这里学习使用的大模型是 deepseek，毕竟成本摆在这里，它的 token 很便宜。<br/>

## 一.大模型请求参数
通过 Http 向大模型发起请求时，有一些必选的参数，还有一些重要的可选的参数，这里作简单介绍。<br/>

访问大模型时，以 JSON 形式传递请求参数。<br/>

```
{
  "model": "deepseek-chat",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Who are you?"},
    {"role": "assistant", "content": "Can I help you?"}
  ],
  "stream": true,
  "enable_search": true
}
```
### 1.role
system：给大模型设置的人设，大模型的响应都会遵循这个设定。<br/>
user：用户的输入，也就是网页上我们熟悉的对话框。<br/>
assistant：大模型的响应数据，通常用来实现上下文。<br/>

### 2.stream
true：流式调用（非阻塞调用）。<br/>
false：一次性响应（阻塞调用）。<br/>

### 3.enable_search
true：开始联网搜索功能，大模型会根据搜索结果作为参考信息。<br/>
false：关闭。<br/>

## 二.LangChain4j会话功能
### 1.快速入门
#### 1）引入依赖
```
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
    <version>1.0.1</version>
</dependency>
```
#### 2）构建 OpenAiChatModel对象
```java
OpenAiChatModel model = OpenAiChatModel.builder().
    .baseUrl("https://api.deepseek.com")
    .apiKey(System.getenv("API_KEY"))
    .modelName("deepseek-chat")
    .logResponses(true)  // 开启响应日志
    .logRequests(true)   // 开启请求日志
    .build();
```
System.getenv("API_KEY") 会获取系统环境变量中名为 API_KEY 的变量。<br/>
#### 3）调用chat方法与模型交互
```java
String result = model.chat("who are you?");
System.out.println(result);
```

### 2.LangChain4j集成Spring
#### 1）构建SpringBoot项目
#### 2）引入起步依赖
```
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai-spring-boot-starter</artifactId>
    <version>1.0.1-beta6</version>
</dependency>
```
spring的版本是：3.5.0<br/>
#### 3）application.yml中配置大模型
```
langchain4j:
  open-ai:
    chat-model:
      base-url: "https://api.deepseek.com"
      api-key: ${API_KEY}
      model-name: deepseek-chat
```
#### 4）开发接口，调用大模型
```java
package cn.edu.bjut.ainote.controller;

import dev.langchain4j.model.openai.OpenAiChatModel;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ChatController {

    @Autowired
    private OpenAiChatModel openAiChatModel;

    @RequestMapping("/chat")
    public String chat(String message) {
        String result = openAiChatModel.chat(message);
        return result;
    }
}
```
### 3.AIService工具类
封装了一些 model 的复杂功能。<br/>

#### 1）引入依赖
```
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-spring-boot-starter</artifactId>
    <version>1.0.1-beta6</version>
</dependency>
```
#### 2）声明接口
```java
package cn.edu.bjut.ainote.service;

public interface AiNoteService {

    /**
     * 聊天方法
     * @param msg
     * @return
     */
    public String chat(String msg);
}
```
#### 3）使用AiService为接口创建代理对象
AiServices采取基于动态代理+声明式注解两种模式来调用方法。<br/>
在这里先介绍基于动态代理的方法。<br/>

动态代理是在运行时动态创建代理类和对象的技术，不需要预先编写具体的实现类。<br/>

将 AiNoteService 交给 Spring 容器管理，使用时注入即可。<br/>
```java
@Configuration
public class CommonConfig {

    @Autowired
    private OpenAiChatModel openAiChatModel;

    @Bean
    public AiNoteService aiNoteService() {
        AiNoteService aiNoteService = AiServices.builder(AiNoteService.class)
                .chatModel(openAiChatModel)
                .build();

        return aiNoteService;
    }
}
```
#### 4）在Controller中注入并使用
```java
@Autowired
private AiNoteService aiNoteService;

@RequestMapping("/chat")
public String chat(String message) {
    String result = aiNoteService.chat(message);
    return result;
}
```
### 4.AiService工具类-声明式注解
在这里我们介绍一下另一种调用方法的模式--声明式注解。<br/>

声明式注解给我们提供了两种使用方案：1）自动装配；2）手动装配。<br/>
我们只需要修改注解的一个参数即可。<br/>
```java
// 手动装配
@AiService(
        wiringMode = AiServiceWiringMode.EXPLICIT,       // 手动装配，默认为自动装配
        chatModel = "openAiChatModel"
)

// 自动装配
@AiService()
public interface AiNoteService {

    /**
     * 聊天方法
     * @param msg
     * @return
     */
    public String chat(String msg);
}
```

### 5.流式调用
#### 1）引入依赖
为了能够正确接收并处理流式调用下的响应，我们要引入两个新的依赖。
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-reactor</artifactId>
    <version>1.0.1-beta6</version>
</dependency>
```
#### 2）配置流式模型对象
```java
langchain4j:
  open-ai:
    chat-model:
      base-url: "https://api.deepseek.com"
      api-key: sk-2b73b5fe82c94e168a14f138b7719010
      model-name: deepseek-chat
    streaming-chat-model:
      base-url: "https://api.deepseek.com"
      api-key: sk-2b73b5fe82c94e168a14f138b7719010
      model-name: deepseek-chat

@AiService(
        wiringMode = AiServiceWiringMode.EXPLICIT,       // 手动装配，默认为自动装配
        chatModel = "openAiChatModel",
        streamingChatModel = "openAiStreamingChatModel"
)
```
#### 3）切换接口中方法的返回值类型
将 String 修改为 Flux<String>，用来接收流式响应数据。
```java
public interface AiNoteService {

    /**
     * 聊天方法
     * @param msg
     * @return
     */
    public Flux<String> chat(String msg);
}
```
#### 4）修改controller中的代码
修改相应的代码，并将编码设置为 utf-8，以免出现乱码。
```java
@RequestMapping(value = "/chat", produces = "text/html;charset=utf-8")
public Flux<String> chat(String message) {
    // 流式调用
    Flux<String> result = aiNoteService.chat(message);
    return result;
}
```

### 6.消息注解
#### 1）@SystemMessage
langchain4j 的 AiService 为我们提供了两个消息注解，@SystemMessage 和 @UserMessage，可以让我们基于注解为会话添加消息。<br/>

如果预设的提示词太长，可以将其写入到外部 txt 文件中，把文件放入到 resourses 目录下，并在注解中指定该外部文件。<br/>

```java
// 直接写明提示词 和 提示词写入到外部文件，再传递给注解。
public interface AiNoteService {

    /**
     * 聊天方法
     * @param msg
     * @return
     */
    @SystemMessage("你是我的好朋友小明，一个魔幻世界中的狡猾的地精商人")
    @SystemMessage(fromResource = "system.txt")
    public Flux<String> chat(String msg);
}
```

#### 2）@UserMessage
@UserMessage 注解会将内容直接附加到用户对话前。<br/>

通常硬性要求使用 {{it}}，但也可以使用 @V 指定其他的变量名。<br/>

```java
public interface AiNoteService {
    @UserMessage("你是我的好朋友小明，{{it}}")
    public Flux<String> chat(String message);
}

public interface AiNoteService {
    @UserMessage("你是我的好朋友小明，{{msg}}")
    public Flux<String> chat(@V("msg") String message);
}
```

### 7.会话记忆
#### 1）定义会话记忆对象
调用一次大模型，只会响应这一次的结果。<br/>

想要让大模型拥有记忆，方法就是把上下文一起发送给大模型。<br/>
```java
@Configuration
public class CommonConfig {

    // 构建会话记忆对象
    @Bean
    public ChatMemory chatMemory() {
        MessageWindowChatMemory chatMemory = MessageWindowChatMemory.builder()
                .maxMessages(20)
                .build();
        return chatMemory;
    }
}

```
#### 2）配置会话记忆对象
```java
package cn.edu.bjut.ainote.service;

@AiService(
        wiringMode = AiServiceWiringMode.EXPLICIT,       // 手动装配，默认为自动装配
        chatModel = "openAiChatModel",
        streamingChatModel = "openAiStreamingChatModel",
        chatMemory = "chatMemory"
)
```

### 8.会话记忆隔离
#### 1）定义会话记忆对象提供者
像前一节一样，使用 MessageWindowChatMemory 来创建会话记忆对象，但是我们这次使用 memoryId 来创建。<br/>

提供者允许我们延迟创建对象。<br/>
```java
package cn.edu.bjut.ainote.config;

@Configuration
public class CommonConfig {

    // 构建会话记忆对象提供者
    @Bean
    public ChatMemoryProvider chatMemoryProvider() {
        ChatMemoryProvider chatMemoryProvider = new ChatMemoryProvider() {
            @Override
            public ChatMemory get(Object memoryId) {
                return MessageWindowChatMemory.builder()
                        .maxMessages(20)
                        .chatMemoryStore(chatMemoryStore)
                        .id(memoryId)
                        .build();
            }
        };

        return chatMemoryProvider;
    }
}
```
#### 2）配置会话记忆对象提供者
```java
@AiService(
        wiringMode = AiServiceWiringMode.EXPLICIT,       // 手动装配，默认为自动装配
        chatModel = "openAiChatModel",
        streamingChatModel = "openAiStreamingChatModel",
        chatMemory = "chatMemory",
        chatMemoryProvider = "chatMemoryProvider"
)
```
#### 3）接口方法中添加参数 memoryId
注解 @MemoryId 用来标志该参数为会话ID。<br/>

注解 @UserMessage 用来标志该参数为用户输入<br/>
```java
public interface AiNoteService {

    /**
     * 聊天方法
     * @param msg
     * @return
     */
    @SystemMessage(fromResource = "system.txt")
    public String chat(@MemoryId String memoryId, @UserMessage String msg);
}
```
#### 4）Controller 中 chat 接口接收 memoryId
```java
@RestController
public class ChatController {

    @Autowired
    private AiNoteService aiNoteService;

    @RequestMapping(value = "/chat", produces = "text/html;charset=utf-8")
    public String[] chat(String memoryId, String message) {
        String result = aiNoteService.chat(memoryId,message);
        String[] split = result.split("\\|");
        return split;
    }
}
```
#### 5）前端页面请求时传递 memoryId

### 9.会话记忆持久化
#### 1）准备 Redis 环境
#### 2）引入 Redis 起步依赖
```
<!-- redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
#### 3）配置 Redis 连接信息
```
spring:
  data:
    redis:
      database: 0
      host: localhost
      port: 6379
```
#### 4）提供 ChatMemoryStore 实现类
```java
package cn.edu.bjut.ainote.repository;

import java.time.Duration;
import java.util.List;

@Repository
public class RedisChatMemoryStore implements ChatMemoryStore {

    // 注入 RedisTemplate
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public List<ChatMessage> getMessages(Object memoryId) {
        // 获取会话消息
        String json = stringRedisTemplate.opsForValue().get(memoryId);
        // 把json字符串转化为List<ChatMessage>
        List<ChatMessage> list = ChatMessageDeserializer.messagesFromJson(json);
        return list;
    }

    @Override
    public void updateMessages(Object memoryId, List<ChatMessage> list) {
        // 更新会话消息
        // 1.把list转化为json数据
        String json = ChatMessageSerializer.messagesToJson(list);
        // 2.把json数据存储到redis中
        stringRedisTemplate.opsForValue().set(memoryId.toString(),json, Duration.ofDays(1));

    }

    @Override
    public void deleteMessages(Object memoryId) {
        stringRedisTemplate.delete(memoryId.toString());
    }
}

```
#### 5）配置 ChatMemoryStore
```java
@Configuration
public class CommonConfig {

    @Autowired
    private ChatMemoryStore chatMemoryStore;

    // 构建会话记忆对象提供者
    @Bean
    public ChatMemoryProvider chatMemoryProvider() {
        ChatMemoryProvider chatMemoryProvider = new ChatMemoryProvider() {
            @Override
            public ChatMemory get(Object memoryId) {
                return MessageWindowChatMemory.builder()
                        .maxMessages(20)
                        .chatMemoryStore(chatMemoryStore)
                        .id(memoryId)
                        .build();
            }
        };

        return chatMemoryProvider;
    }
}
```
### 10.RAG快速入门
RAG 分为两部分，一是如何将数据存储到向量数据库，二是如果从向量数据库中检索数据。
#### 1）引入依赖
```
<!-- ease rag -->
<dependency>
  <groupId>dev.langchain4j</groupId>
  <artifactId>langchain4j-easy-rag</artifactId>
  <version>1.0.1-beta6</version>
</dependency>
```
#### 2）加载知识数据文档
#### 3）构建向量数据库操作对象
```java
@Configuration
public class CommonConfig {

    /**
     * 构建向量数据库操作对象
     * @return
     */
    @Bean
    public EmbeddingStore store() {
        // 1.加载文档进内存
        List<Document> documents = ClassPathDocumentLoader.loadDocuments("content");
        // 2.构建向量数据库操作对象
        InMemoryEmbeddingStore store = new InMemoryEmbeddingStore();
        // 3.构建一个 EmbeddingStoreIngestor 对象，完成文本数据切割，向量化，存储
        EmbeddingStoreIngestor ingestor = EmbeddingStoreIngestor.builder()
                .embeddingStore(store)
                .build();

        ingestor.ingest(documents);
        return store;
    }
}
```
#### 4）把文档切割、向量化并存储到向量数据库中
在快速入门这一节中，这部分操作 langchain4j 已经为我们封装好了。
#### 5）构建向量数据库检索对象
```java
@Configuration
public class CommonConfig {

    /**
     * 构建向量数据库检索对象
     * @param store
     * @return
     */
    @Bean
    public ContentRetriever contentRetriever(EmbeddingStore store) {
        return EmbeddingStoreContentRetriever.builder()
                .embeddingStore(store)
                .minScore(0.5)
                .maxResults(3)
                .build();
    }
}
```
#### 6）配置向量数据库检索对象
```java
@AiService(
        wiringMode = AiServiceWiringMode.EXPLICIT,       // 手动装配，默认为自动装配
        chatModel = "openAiChatModel",
        streamingChatModel = "openAiStreamingChatModel",
        chatMemory = "chatMemory",
        chatMemoryProvider = "chatMemoryProvider",
        contentRetriever = "contentRetriever"
)
```

### 11.RAG知识库-核心API
rag 技术的实现流程可以分为：Document Loader（文档加载器） -> Document Parser（文档解析器） -> Document Splitter（文档分割器） -> EmbeddingModel（向量模型） -> EmbeddingStore（向量数据库操作对象）。
#### 1）文档加载器
从不同来源获取外部文档，将其加载到内存中。
```java
// 相对于类路径加载
List<Document> documents = ClassPathDocumentLoader.loadDocuments("content");
// 根据磁盘绝对路径加载
List<Document> documents = FileSystemDocumentLoader.loadDocuments("E://");
// 根据url路径加载
List<Document> documents = UrlDocumentLoader.load("http://");
// ...
```
#### 2）文本解析器
对加载到内存中不同类型的文档进行解析，把非纯文本转化为纯文本。
```java
// 引入依赖
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-document-parser-apache-pdfbox</artifactId>
    <version>1.0.1-beta6</version>
</dependency>

// TextDocumentParser，解析纯文本格式的文件
// ApachePdfBoxDocumentParser，解析pdf格式文件
// ApachePoiDocumentParser，解析微软的office文件，例如DOC、PPT、XLS
// ApacheTikaDocumentParser（默认），几乎可以解析所有格式的文件
List<Document> documents = ClassPathDocumentLoader.loadDocuments("content"，new ApachePdfBoxDocumentParser());
```
#### 3）文本分割器
对解析后的文档进行分割，切割成一个一个的小片段。<br/>

DocumentSplitters.recursive()（默认），递归分割器，优先段落分割，再按照行分割，再按照句子分割，再按照词分割。<br/>

```java
@Configuration
public class CommonConfig {

    /**
     * 构建向量数据库操作对象
     * @return
     */
    @Bean
    public EmbeddingStore store() {
        // 1.加载文档进内存
        List<Document> documents = ClassPathDocumentLoader.loadDocuments("content");
        // 2.构建向量数据库操作对象
        InMemoryEmbeddingStore store = new InMemoryEmbeddingStore();
        
        // 构建文档切割器对象
        DocumentSplitter ds = DocumentSplitters.recursive(500,100);
        // 3.构建一个 EmbeddingStoreIngestor 对象，完成文本数据切割，向量化，存储
        EmbeddingStoreIngestor ingestor = EmbeddingStoreIngestor.builder()
                .embeddingStore(store)
                .documentSplitter(ds)
                .build();

        ingestor.ingest(documents);
        return store;
    }
}

```
#### 4）向量模型
用于把切割后的片段向量化。

```java
// 
langchain4j:
  open-ai:
    embedding-model:
      base-url: 
      api-key: 
      model-name: 
      log-responses: true
      log-requests: true

@Configuration
public class CommonConfig {

    @Autowired
    private EmbeddingModel embeddingModel;

    /**
     * 构建向量数据库操作对象
     * @return
     */
    @Bean
    public EmbeddingStore store() {
        // 1.加载文档进内存
        List<Document> documents = ClassPathDocumentLoader.loadDocuments("content");
        // 2.构建向量数据库操作对象
        InMemoryEmbeddingStore store = new InMemoryEmbeddingStore();

        // 构建文档切割器对象
        DocumentSplitter ds = DocumentSplitters.recursive(500,100);
        // 3.构建一个 EmbeddingStoreIngestor 对象，完成文本数据切割，向量化，存储
        EmbeddingStoreIngestor ingestor = EmbeddingStoreIngestor.builder()
                .embeddingStore(store)
                .documentSplitter(ds)
                .embeddingModel(embeddingModel)  // 在这里完成配置
                .build();

        ingestor.ingest(documents);
        return store;
    }

    /**
     * 构建向量数据库检索对象
     * @param store
     * @return
     */
    @Bean
    public ContentRetriever contentRetriever(EmbeddingStore store) {
        return EmbeddingStoreContentRetriever.builder()
                .embeddingStore(store)
                .minScore(0.5)
                .maxResults(3)
                .embeddingModel(embeddingModel)  // 在这里完成配置
                .build();
    }
}
```
### 5）向量数据库操作对象
用于操作向量数据库（添加、检索）
```java
// 引入依赖
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-community-redis-spring-boot-starter</artifactId>
    <version>1.0.1-beta6</version>
</dependency>

// 配置向量数据库信息
langchain4j:
  open-ai:
    community:
      redis:
        host:localhost
        port:6379

// 注入 RedisEmbeddingStore 并使用
@Configuration
public class CommonConfig {

    @Autowired
    private EmbeddingModel embeddingModel;

    @Autowired
    private RedisEmbeddingStore redisEmbeddingStore;

    /**
     * 构建向量数据库操作对象
     * @return
     */
    @Bean
    public EmbeddingStore store() {
        // 1.加载文档进内存
        List<Document> documents = ClassPathDocumentLoader.loadDocuments("content");

        // 构建文档切割器对象
        DocumentSplitter ds = DocumentSplitters.recursive(500,100);

        // 3.构建一个 EmbeddingStoreIngestor 对象，完成文本数据切割，向量化，存储
        EmbeddingStoreIngestor ingestor = EmbeddingStoreIngestor.builder()
                .embeddingStore(redisEmbeddingStore)  // 在这里完成配置
                .documentSplitter(ds)
                .embeddingModel(embeddingModel)
                .build();

        ingestor.ingest(documents);
        return store;
    }

    /**
     * 构建向量数据库检索对象
     * @param store
     * @return
     */
    @Bean
    public ContentRetriever contentRetriever() {
        return EmbeddingStoreContentRetriever.builder()
                .embeddingStore(redisEmbeddingStore)  // 在这里完成配置
                .minScore(0.5)
                .maxResults(3)
                .embeddingModel(embeddingModel)
                .build();
    }
}
```
### 12.Function Calling
大模型根据用户请求，调用外部函数或api来完成特定任务。<br/>

这部分 langchain4j 为我们完成了大部分流程的封装，我们只需要写好提示信息即可。<br/>

注解 @Tool 对方法的作用进行描述。<br/>
注解 @P 对方法的参数进行描述。<br/>

```java
// 准备工具方法
@Component
public class CommonTool{
    @Autowired
    private Service service;
    @Tool("添加用户基本信息")
    @P("姓名") String name;
    @P("性别") String gender;
    @P("电话") String phone;
    service.insert(new Person(name,gender,phone);
}

// 配置工具方法
@AiService(
        wiringMode = AiServiceWiringMode.EXPLICIT,       // 手动装配，默认为自动装配
        chatModel = "openAiChatModel",
        streamingChatModel = "openAiStreamingChatModel",
        chatMemory = "chatMemory",
        chatMemoryProvider = "chatMemoryProvider",
        contentRetriever = "contentRetriever",
        tools = "commonTool"
```
