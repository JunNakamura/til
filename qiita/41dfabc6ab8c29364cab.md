---
title: jerseyの@QueryParamと@PathParamはURLデコードを自動でする
tags: Java jersey urlエンコード token
author: n_slender
slide: false
---
jersyでは、下記のような@QueryParamや@PathParamなどで受け取った値は、標準では自動でURLデコード処理がかけられています。

このことは[javadoc](https://jersey.java.net/apidocs/2.22/jersey/javax/ws/rs/QueryParam.html) にももちろん書かれています。

```
@GET
public Response findByName(@QueryParam("name") String name) {
　　　// このときnameという変数はURLデコード済みの状態
　　　/* 中略 */
}
```

URLデコード処理をして欲しくないときは[@Encoded](https://jersey.java.net/apidocs/2.22/jersey/javax/ws/rs/Encoded.html) のアノテーションを追加で付与すればよいです。このアノテーションはclassにもつけられ、その場合はそのクラスの全てのメソッドの全てのパラメタに適用されます。

```
@GET
public Response findByName(@QueryParam("name") @Encoded String name) {
　　　// このときnameはリクエストされたURLのままの状態
　　　/* 中略 */
}
```

暗号化されたトークンなど、そのままの状態で扱いたい場合は、上記のようにデコード処理をさせないようにするか、[URLセーフ](http://qiita.com/n_slender/items/13bb4196bab273e081c7)なものにするかです。

また、[Json Web Tokens](https://jwt.io/) でも、URLセーフなトークンを実現できます。


#　参考サイト

http://stackoverflow.com/questions/13626990/jax-rs-automatic-decode-pathparam

http://stackoverflow.com/questions/2291428/jax-rs-pathparam-how-to-pass-a-string-with-slashes-hyphens-equals-too

http://qiita.com/kaiinui/items/21ec7cc8a1130a1a103a


