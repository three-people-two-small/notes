# Retrofit介绍

Retrofit是一个http框架，它主要是对OkHttp做了一层封装，使用起来更加方便，整个封装过程如下：  
应用层 -> Retrofit -> okhttp -> http  

## Retrofit和OkHttp的比较  

1. 从整个封装过程可以看出，OkHttp是对http协议封装的一套请求的客户端，而Retrofit是  对OkHttp做的一层封装，简化了代码的调用过程，使得开发更加容易  

2. Retrofit在OkHttp的基础之上，对我们怎么调用网络请求接口，以及从接口拿到数据后如何处理做了比较成熟的处理。

## 如何使用

请求

```java
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```

生成retrofit

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

调用

```java
Call<List<Repo>> repos = service.listRepos("octocat");
```

从官网的简单例子入手，可以看出整个使用过程非常的简单，首先，通过接口的方式定义一个http请求，请求的参数通过注解的方式定义；然后，构建一个retrofit客户端，由retrofit客户端生成请求接口的代理对象；最后，由代理对象调用请求方法，获取调用结果。  

使用retrofit非常的简单，所有调用http的细节，都不需要我们关心，我们只需要用我们习惯的java方式来实现调用。具体调用细节，retrofit已经帮我们封装好了。

## 注解

### 请求方法类

 1. get请求 @GET + @Query + @QueryMap

 ```java
/**
 * 请求方法注解之@GET+@Query+@Path
 *
 * @param test path路径
 * @param name query参数name
 * @param id   query参数id
 * @return 响应结果
 */
@GET("/api/{test}")
Call <ResponseBody> getForQuary(@Path("test") String test, @Query("name") String name, @Query("id") int id);
```

```java
/**
 * 请求方法注解之@GET+@QueryMap+@Path
 *
 * @param test path路径
 * @param map map
 * @return 响应结果
 */
@GET("/api/{test}")
Call <ResponseBody> getForQuaryMap(@Path("test") String test, @QueryMap Map<String, String> map);
```

```java
/**
 * 静态请求
 */
@GET("/api/test?name=xuwei&id=10")
Call<ResponseBody> get();
```

2. post表单请求 @POST + @FormUrlEncoded + @Field + @FieldMap

```java
/**
 * 请求方法注解之@POST+@FormUrlEncoded+@Path+@Field
 *
 * @param name 参数name
 * @param id   参数id
 * @return 请求结果
 */
@POST("/api/{test}")
@FormUrlEncoded
Call <ResponseBody> postForForm(@Path("test") String test, @Field("name") String name, @Field("id") int id);
```

```java
/**
 * 请求方法注解之@POST+@FormUrlEncoded+@Path+@FieldMap
 *
 * @param map 参数map
 * @return 请求结果
 */
@POST("/api/{test}")
@FormUrlEncoded
Call <ResponseBody> postForFormMap(@Path("test") String test, @FieldMap Map <String, String> map);
```

```java
/**
 * 请求方法注解之@POST+@Path+@Body
 *
 * @param path        path
 * @param requestBody requestBody
 * @return 请求结果
 */
@POST("/api/{path}")
Call <ResponseBody> post(@Path("path") String path, @Body RequestBody requestBody);
```

注意，对于POST请求，@Body注解不能加@FormUrlEncoded不然会报错。

## convert转换器

对于retrofit默认情况下如果我们使用requestBody和responseBody就能实现发送请求和接受响应，但是，如果我们想通过自己定义的实体对象来发送数据和接受数据，retrofit也提供了方法给我们使用，那就是自定义convert转换器。  

1. 自定义转换器的使用  

POST+body请求

```java
/**
 * 自定义请求参数和自定义返回参数
 *
 * @param path   path
 * @param params 自定义请求参数
 * @return 自定义返回参数
 */
@POST("/api/{path}")
Call <MyResponse> postBody(@Path("path") String path, @Body Params params);
```

自定义转换器

 ```java
package com.xw.retrofit;

import com.alibaba.fastjson.JSONObject;
import com.xw.web.MyResponse;
import com.xw.web.Params;
import okhttp3.MediaType;
import okhttp3.RequestBody;
import okhttp3.ResponseBody;
import retrofit2.Converter;
import retrofit2.Retrofit;

import java.io.IOException;
import java.lang.annotation.Annotation;
import java.lang.reflect.Type;

/**
 * create by 山人二小 on 2018/11/11
 */
public class MyConverter extends Converter.Factory {

@Override
public Converter <ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations,
Retrofit retrofit) {
if (type == MyResponse.class) {
return MyResponseBodyConverter.INSTANCE;
}
return null;
}

 public Converter <?, RequestBody> requestBodyConverter(Type type,
   Annotation[] parameterAnnotations, Annotation[] methodAnnotations, Retrofit retrofit) {
		if (type == Params.class){
			return MyRequestBodyConverter.INSTANCE;
		}
		return null;
	}

	static final class MyRequestBodyConverter implements Converter<Params, RequestBody> {
		static final MyConverter.MyRequestBodyConverter INSTANCE = new MyConverter.MyRequestBodyConverter();
		@Override
		public RequestBody convert(Params params) {
			String json = JSONObject.toJSONString(params);
			RequestBody requestBody =  RequestBody.create(MediaType.parse("application/json; charset=utf-8"), json);
			return requestBody;
		}
	}

	static final class MyResponseBodyConverter
			implements Converter <ResponseBody, MyResponse> {
		static final MyConverter.MyResponseBodyConverter INSTANCE = new MyConverter.MyResponseBodyConverter();

		@Override
		public MyResponse convert(ResponseBody value) throws IOException {
			String result = value.string();
			MyResponse myResponse = JSONObject.parseObject(result, MyResponse.class);
			return myResponse;
		}
	}
}
 ```

 
