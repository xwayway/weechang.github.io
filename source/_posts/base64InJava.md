---
title: Base64在Java中的应用
date: 2019-04-25 11:28:14
categories: 日常记录
tags: [日常记录, Base64, 编码，Java]
---

>Base64是一种能将任意Binary资料用64种字元组合成字串的方法，而这个Binary资料和字串资料彼此之间是可以互相转换的，十分方便。在实际应用上，Base64除了能将Binary资料可视化之外，也常用来表示字串加密过后的内容。

<!-- more -->

最近做到一个项目，请求是支付宝的H5付款地址，完整URL类似于 

```javascript
https://abc.com/?payUrl=https://open.alipay.com/gateway.do?sing=xxxxx&bizContent={"appId":"2018xxxx","subject":"苹果X"}
```

这个时候，通过 request.getParameter("payUrl"); 方法获取参数就会有问题。应该说是直接访问接口400。第一直觉是因为参数是URL导致的。于是便想要通过将参数编码后再参数，刚开始想的是使用URL编码，接口400的情况不存在了，但是解码后会有问题，因为还会被浏览器二次URL编码。

针对二次URL编码的情况，想的是二次解码就行了，试了下貌似不行。后来开始换方案，直接换成Base64编码传参。


在Java中Base64方案有以下种

## JAVA8 之前的方法

在低于Java8的版本中做Base64的编码与解码，可以使用JDK里sun.misc下的BASE64Encoder和BASE64Decoder这两个类，具体用法如下：

```java
final BASE64Encoder encoder = new BASE64Encoder();
final BASE64Decoder decoder = newBASE64Decoder();
final String text ="weechang千刀";
final byte[] textByte = text.getBytes("UTF-8");
//编码
final String encodedText = encoder.encode(textByte);
System.out.println(encodedText);
//解码
System.out.println(new String(decoder.decodeBuffer(encodedText),"UTF-8"));
final BASE64Encoder encoder = new BASE64Encoder();
final BASE64Decoder decoder = new BASE64Decoder();
final String text ="weechang千刀";
final byte[] textByte = text.getBytes("UTF-8");
//编码
final String encodedText =encoder.encode(textByte);
System.out.println(encodedText);
//解码
System.out.println(new String(decoder.decodeBuffer(encodedText),"UTF-8"));
```

只是这个sun.misc所提供的Base64功能，编码和解码的效率并不太好，而且在以后的Java版本可能就不被支持了，不是很建议使用。

## Apache Commons Codec 方法
   
Apache Commons Codec有提供Base64的编码与解码功能，使用org.apache.commons.codec.binary下的Base64类具体用法如下：
   
```java 
final Base64 base64 = new Base64();
final String text = "weechang千刀";
final byte[] textByte = text.getBytes("UTF-8");
//编码
final String encodedText = base64.encodeToString(textByte);System.out.println(encodedText);
//解码
System.out.println(new String(base64.decode(encodedText),"UTF-8"));
final Base64 base64 = new Base64();
final String text = "weechang千刀";
final byte[] textByte = text.getBytes("UTF-8");
//编码
final String encodedText = base64.encodeToString(textByte);System.out.println(encodedText);
//解码
System.out.println(new String(base64.decode(encodedText), "UTF-8"));
```
以上的代码看起来比用sun.misc还要更精简，效能实际执行起来也快了不少。缺点是需要引用Apache Commons Codec，很麻烦。

### Java8 方法

Java 8的java.util套件中，新增了Base64的类别，可以用来处理Base64的编码与解码，用法如下：

```java
final Base64.Decoder decoder = Base64.getDecoder();
final Base64.Encoder encoder = Base64.getEncoder();
final String text = "weechang千刀";
final byte[] textByte = text.getBytes("UTF-8");
//编码
final String encodedText = encoder.encodeToString(textByte);
System.out.println(encodedText);
//解码
System.out.println(new String(decoder.decode(encodedText), "UTF-8"));
finalBase64.Decoder decoder = Base64.getDecoder();
final Base64.Encoder encoder = Base64.getEncoder();
final String text = "weechang千刀";
final byte[] textByte = text.getBytes("UTF-8");
//编码
final String encodedText = encoder.encodeToString(textByte);
System.out.println(encodedText);
//解码
System.out.println(new String(decoder.decode(encodedText), "UTF-8"));
```
   
与sun.misc和Apache Commons Codec所提供的Base64编解码器来比较的话，Java 8提供的Base64拥有更好的效能。

但是实测后，发现通过编码后的Base64传到服务端不能进行解码。后来经过字符串比较才发现，编码的Base64和解码的Base64有一部分数据是不一致的。Base64中的“+”到了服务端会被" "替换掉。所以得使用Java8中的另外一种Base64方案。

### Java8 URL 的 Base64

Java8 针对需要在URL中传输的Base64提供了另外一套解决方法，具体用法如下：

```java 
final Base64.Decoder decoder = Base64.getUrlDecoder();
final Base64.Encoder encoder = Base64.getUrlEncoder();
final String text = "weechang千刀";
final byte[] textByte = text.getBytes("UTF-8");
//编码
final String encodedText = encoder.encodeToString(textByte);
System.out.println(encodedText);
//解码
System.out.println(new String(decoder.decode(encodedText), "UTF-8"));
finalBase64.Decoder decoder = Base64.getDecoder();
final Base64.Encoder encoder = Base64.getEncoder();
final String text = "weechang千刀";
final byte[] textByte = text.getBytes("UTF-8");
//编码
final String encodedText = encoder.encodeToString(textByte);
System.out.println(encodedText);
//解码
System.out.println(new String(decoder.decode(encodedText), "UTF-8"));
```

通过以上的方法编码后的Base64字符串在URL传输中，是安全的，去除了会被URL特殊处理的字符。

至此，绕路一大圈，问题终于得到解决。

