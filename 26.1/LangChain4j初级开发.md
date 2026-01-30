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
