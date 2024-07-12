# 【仕様理解】
## １．httpセッションの使い方
#### HTTPセッションを抽象化した機能を提供する
> ### ExecutionContext
> **public ExecutionContext setSessionScopedVar(java.lang.String varName, java.lang.Object varValue)**  
> セッションスコープ上の変数の値を設定する。  
> 既に定義済みの変数は上書きされる。

> ### HttpSessionStore
> **public void save(java.lang.String sessionId,
>                 java.util.List<SessionEntry> entries,
>                 ExecutionContext executionContext)**  
> セッションの内容をストアに保存する。


## ２．スコープの使い分け
#### リクエストスコープ / セッションスコープ
#### DBストア / HTTPセッションストア
> セッションを識別するためにセッションIDを発行し、 クッキー( NABLARCH_SID (変更可))を使用して、セッションを追跡する。 そして、セッションIDごとにセッションストアと呼ばれる保存先へ読み書きする機能を提供する。  
> 本機能では、セッションIDごとにセッションストアに読み書きされる値をセッション変数と呼ぶ。


１．https://nablarch.github.io/docs/5u18/javadoc/nablarch/fw/ExecutionContext.html  
２．https://nablarch.github.io/docs/5u18/javadoc/nablarch/common/web/session/SessionUtil.html  
３．https://nablarch.github.io/docs/5u18/javadoc/nablarch/common/web/session/store/HttpSessionStore.html
