---
title: 简单的加密算法-MD5、SHA
date: 2018-09-02 16:14:11
categories: 算法
tags: [md5, sha1]
photos: 
---

### 摘要

简单的介绍常用的两个算法及代码实现

### MD5算法

**MD5消息摘要算法**（英语：MD5 Message-Digest Algorithm），一种被广泛使用的[密码散列函数](https://baike.baidu.com/item/%E5%AF%86%E7%A0%81%E6%95%A3%E5%88%97%E5%87%BD%E6%95%B0)，可以产生出一个128位16字节的散列值（hash value），用于确保信息传输完整一致。 摘自[百度百科](https://baike.baidu.com/item/MD5/212708?fromtitle=MD5算法&fromid=174909&fr=aladdin)

<!-- more --> 

#### 安全性

MD5算法因其不可逆的设计，理论上是不可破解的，但是某些网站将常用密码经md5算法加密后存储到数据库中进行匹配，从而实现事实上的破解。所以现在大多数网站强制用户使用数字大小写字母等复杂组合密码，以此提高用户密码的安全性。

#### java代码实现

```java
package util;

import java.security.MessageDigest;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import java.util.SortedMap;

// MD5加密工具类
public class MD5Util {

	// 创建md5摘要,规则是:按参数名称a-z排序,遇到空值的参数不参加签名。
	public static String createSign(SortedMap<String, String> packageParams,
			String appid, String key) {
		StringBuffer sb = new StringBuffer();
		Set es = packageParams.entrySet();
		Iterator it = es.iterator();
		while (it.hasNext()) {
			Map.Entry entry = (Map.Entry) it.next();
			String k = (String) entry.getKey();
			String v = (String) entry.getValue();
			if (null != v && !"".equals(v) && !"sign".equals(k)
					&& !"key".equals(k)) {
				sb.append(k + "=" + v + "&");
			}
		}
		sb.append("key=" + key);
		String sign = MD5Encode(sb.toString(), "UTF-8").toUpperCase();
		return sign;
	}

	private static String byteArrayToHexString(byte b[]) {
		StringBuffer resultSb = new StringBuffer();
		for (int i = 0; i < b.length; i++)
			resultSb.append(byteToHexString(b[i]));

		return resultSb.toString();
	}

	private static String byteToHexString(byte b) {
		int n = b;
		if (n < 0)
			n += 256;
		int d1 = n / 16;
		int d2 = n % 16;
		return hexDigits[d1] + hexDigits[d2];
	}

	public static String MD5Encode(String origin, String charsetname) {
		String resultString = null;
		try {
			resultString = new String(origin);
			MessageDigest md = MessageDigest.getInstance("MD5");
			if (charsetname == null || "".equals(charsetname))
				resultString = byteArrayToHexString(md.digest(resultString
						.getBytes()));
			else
				resultString = byteArrayToHexString(md.digest(resultString
						.getBytes(charsetname)));
		} catch (Exception exception) {
		}
		return resultString;
	}

	private static final String hexDigits[] = { "0", "1", "2", "3", "4", "5",
			"6", "7", "8", "9", "a", "b", "c", "d", "e", "f" };
}
```

### SHA算法

**安全散列算法**（英语：Secure Hash Algorithm，缩写为SHA）是一个[密码散列函数](https://baike.baidu.com/item/%E5%AF%86%E7%A0%81%E6%95%A3%E5%88%97%E5%87%BD%E6%95%B0)家族，是[FIPS](https://baike.baidu.com/item/FIPS)所认证的安全[散列算法](https://baike.baidu.com/item/%E6%95%A3%E5%88%97%E7%AE%97%E6%B3%95)。能计算出一个数字消息所对应到的，长度固定的字符串（又称消息摘要）的算法。且若输入的消息不同，它们对应到不同字符串的机率很高 。

#### java代码实现

```java
package utils;

import java.security.MessageDigest;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import java.util.SortedMap;

// SHA1算法工具类
public class Sha1Util {

	// api说明： createSHA1Sign创建签名SHA1 getSha1()Sha1签名
	// 创建签名SHA1
	public static String createSHA1Sign(SortedMap<String, String> signParams) throws Exception {
		StringBuffer sb = new StringBuffer();
		Set es = signParams.entrySet();
		Iterator it = es.iterator();
		while (it.hasNext()) {
			Map.Entry entry = (Map.Entry) it.next();
			String k = (String) entry.getKey();
			String v = (String) entry.getValue();
			sb.append(k + "=" + v + "&");
			// 要采用URLENCODER的原始值！
		}
		String params = sb.substring(0, sb.lastIndexOf("&"));

		return getSha1(params);
	}

	// Sha1签名
	public static String getSha1(String str) {
		if (str == null || str.length() == 0) {
			return null;
		}
		char hexDigits[] = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f' };

		try {
			MessageDigest mdTemp = MessageDigest.getInstance("SHA1");
			mdTemp.update(str.getBytes("UTF-8"));

			byte[] md = mdTemp.digest();
			int j = md.length;
			char buf[] = new char[j * 2];
			int k = 0;
			for (int i = 0; i < j; i++) {
				byte byte0 = md[i];
				buf[k++] = hexDigits[byte0 >>> 4 & 0xf];
				buf[k++] = hexDigits[byte0 & 0xf];
			}
			return new String(buf);
		} catch (Exception e) {
			return null;
		}
	}
}
```

### 注：

代码摘自CSDN博客。