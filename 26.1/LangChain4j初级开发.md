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

## 二.LangChain4j快速入门
### 1.引入依赖
```
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
    <version>1.0.1</version>
</dependency>
```
### 2.构建 OpenAiChatModel对象
```java
OpenAiChatModel model = OpenAiChatModel.builder().
    .baseUrl("https://api.deepseek.com")
    .apiKey("")
    .modelName("deepseek-chat")
    .logResponses(true)  // 开启响应日志
    .logRequests(true)   // 开启请求日志
    .build();
```
### 3.调用chat方法与模型交互
```java
String result = model.chat("who are you?");
System.out.println(result);
```

