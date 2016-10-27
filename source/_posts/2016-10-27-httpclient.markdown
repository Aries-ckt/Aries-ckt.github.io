---
layout: post
title: "HttpClient"
date: 2016-10-27 18:57:17 +0800
comments: true
categories: 
---
### HttpClient使用
它不仅是使得客户端发送Http请求变得容易，而且也方便了开发人员测试接口（基于Http协议的），即提高了开发的效率，也方便提高代码的健壮性。因此熟练掌握HttpClient是很重要的必修内容，掌握HttpClient后，相信对于Http协议的了解会更加深入。
### 使用方法
1. 创建HttpClient对象。

```
CloseableHttpClient httpclient = HttpClients.createDefault();
```

2. 创建请求方法的实例，并指定请求URL。如果需要发送GET请求，创建HttpGet对象；如果需要发送POST请求，创建HttpPost对象。

```
 HttpPost httpost = new HttpPost(url);
```

3. 如果需要发送请求参数，可调用HttpGet、HttpPost共同的setParams(HttpParams params)方法来添加请求参数；对于HttpPost对象而言，也可调用setEntity(HttpEntity entity)方法来设置请求参数。

```
List<NameValuePair> nvps = new ArrayList <NameValuePair>();
if(params!=null){//params是方法传过来的参数，是一个map
            	  Set<String> keySet = params.keySet();  
            	  for(String key : keySet) {  
            		  nvps.add(new BasicNameValuePair(key, params.get(key)));  
            	  }  
              }
```


```
if(header!=null&&!header.isEmpty()){//headers是方法传过来的参数，是一个map
                  for(Map.Entry<String,String> entry:header.entrySet()){
                	  httpost.setHeader(entry.getKey(),entry.getValue());
                }
            }
```
如果请求是post,还可以设置参数setEntity

```
httpost.setEntity(new UrlEncodedFormEntity(nvps, charsetName));
```

还可以设置请求和传输超时时间

```
RequestConfig requestConfig = RequestConfig.custom().setSocketTimeout(5000).setConnectTimeout(5000).build();
              httpost.setConfig(requestConfig);
```

4. 调用HttpClient对象的execute(HttpUriRequest request)发送请求，该方法返回一个HttpResponse。

```
 HttpResponse response = httpclient.execute(httpost);
```

5. 调用HttpResponse的getAllHeaders()、getHeaders(String name)等方法可获取服务器的响应头；调用HttpResponse的getEntity()方法可获取HttpEntity对象，该对象包装了服务器的响应内容。程序可通过该对象获取服务器的响应内容。

```
HttpEntity entity = response.getEntity();
StringBuilder sb = new StringBuilder();
BufferedReader red = new BufferedReader(new InputStreamReader(entity.getContent(), charsetName));
String line;
while ((line = red.readLine()) != null) {
    sb.append(line);
    }
EntityUtils.consume(entity);
content = sb.toString();
```

6. 释放连接。无论执行方法是否成功，都必须释放连接

```
catch (Exception e) {
              logger.error(url,e);
              return null;
          } finally {
              //释放连接
              try {
  				httpclient.close();
  			} catch (IOException e) {
  				 logger.error(url,e);
  			}
```

