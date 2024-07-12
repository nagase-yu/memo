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
サーブレットとJSP間の情報受け渡し→ リクエストスコープ  
複数リクエスト間の情報受け渡し → セッションスコープ

#### DBストア / HTTPセッションストア
> セッションを識別するためにセッションIDを発行し、 クッキー( NABLARCH_SID (変更可))を使用して、セッションを追跡する。 そして、セッションIDごとにセッションストアと呼ばれる保存先へ読み書きする機能を提供する。  
> 本機能では、セッションIDごとにセッションストアに読み書きされる値をセッション変数と呼ぶ。

| ストア |格納場所| 特徴 |
|:--------------|-----------------|:-------------|
|DBストア|データベース上のテーブル|・サーバが停止時でもセッション変数の復元が可能<br>・サーバのヒープ領域を圧迫しない。|
|HTTPセッションストア|サーバのヒープ領域|・認証情報の様なアプリケーション全体で頻繁に使用する情報の保持に適している。<br>・画面の入力内容の様な大量データを保存すると、ヒープ領域を圧迫する恐れがある。|

| 用途 | セッションストア |
|:-----------|------------:|
| 入力～確認～完了画面間で入力情報の保持(複数タブでの画面操作を許容しない) |DBストア |
| 入力～確認～完了画面間で入力情報の保持(複数タブでの画面操作を許容する) | HIDDENストア |
| 認証情報の保持 | DBストア or HTTPストア |


###### HTTPセッションストア
サーバ上のHTTPセッションにセッション変数を保持。サーバ負荷軽減のため、軽量な情報に限る。(方式設計書)  
（例）ユーザID, 氏名, 管理者権限有無, ページ番号, 検索条件
```
/**
 * ログインユーザ情報を格納する。
 *
 * @param ctx HTTPリクエストの処理に関連するサーバ側の情報
 * @param userContext ログインユーザ情報
 */
public static void putLoginUserContext(ExecutionContext ctx, LoginUserContext userContext) {
    SessionUtil.put(ctx, LOGIN_USER_CONTEXT_SESSION_KEY, userContext, PUT_STORE_NAME); // PUT_STORE_NAME = "httpSession"
}
```

１．https://nablarch.github.io/docs/5u18/javadoc/nablarch/fw/ExecutionContext.html  
２．https://nablarch.github.io/docs/5u18/javadoc/nablarch/common/web/session/SessionUtil.html  
３．https://nablarch.github.io/docs/5u18/javadoc/nablarch/common/web/session/store/HttpSessionStore.html
