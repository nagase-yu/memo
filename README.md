# 【仕様理解】
## １．httpセッションの使い方
###### HTTPセッションとセッションスコープの違い  
> HTTPセッションとセッションスコープは、それぞれ異なる概念ですが、密接に関連しています。
> 
>  HTTPセッションは、Webサーバーが特定のユーザー（正確には特定のブラウザ）との間で情報を保持するための仕組みです。
> HTTP自体はステートレスなプロトコルであるため、セッションを通じてユーザーの状態を追跡します。  
> HTTPセッションは、ログイン情報、ユーザーの設定、カート内の商品など、ユーザー固有の情報を保持するために使用されます。
>
>  一方、セッションスコープは、HTTPセッション内の特定の領域を指します。  
> セッションスコープに保存されたデータは、そのセッションが有効な間、すなわちユーザーがブラウザを閉じるか、セッションがタイムアウトするまで、複数のリクエストやページ間で利用することができます。
> 
> したがって、HTTPセッションはユーザー固有の情報を保持するための仕組みであり、セッションスコープはそのセッション内でデータを保持するための領域を指します。

###### HTTPセッションを抽象化した機能
> ### ExecutionContext
> **public ExecutionContext setSessionScopedVar(java.lang.String varName, java.lang.Object varValue)**
> 
> セッションスコープ上の変数の値を設定する。  
> 既に定義済みの変数は上書きされる。

> ### SessionUtil  
> **public static void put(ExecutionContext ctx,
>                       java.lang.String name,
>                       java.lang.Object value)**
> 
> SessionStoreに変数を保存する。  
> オブジェクトの保存先はSessionManagerで指定したSessionStoreが選択される

> ### HttpSessionStore
> **public void save(java.lang.String sessionId,
>                 java.util.List<SessionEntry> entries,
>                 ExecutionContext executionContext)**
> 
> セッションの内容をストアに保存する。
* セッションID = セッションストアに読み書きされる値に対応するキー  
* ExecutionContext.setSessionScopedVarだけでは、セッション変数はHTTPセッションとして管理される？

###### まとめ  
HTTPセッションは「仕組み」  
セッションスコープは「情報」  
HTTPセッションストアは「情報のまとまり」


## ２．スコープの使い分け
###### リクエストスコープ / セッションスコープ
サーブレットとJSP間の情報受け渡し→ リクエストスコープ  
複数リクエスト間の情報受け渡し → セッションスコープ

###### DBストア / HTTPセッションストア
> セッションを識別するためにセッションIDを発行し、 クッキー( NABLARCH_SID (変更可))を使用して、セッションを追跡する。 そして、セッションIDごとにセッションストアと呼ばれる保存先へ読み書きする機能を提供する。  
> 本機能では、セッションIDごとにセッションストアに読み書きされる値をセッション変数と呼ぶ。

| ストア |格納場所| 特徴 |
|:--------------|-----------------|:-------------|
|DBストア|データベース上のテーブル|・サーバが停止時でもセッション変数の復元が可能<br>・サーバのヒープ領域を圧迫しない|
|HTTPセッションストア|サーバのヒープ領域|・認証情報の様なアプリケーション全体で頻繁に使用する情報の保持に適している<br>・画面の入力内容の様な大量データを保存すると、ヒープ領域を圧迫する恐れあり|

| 用途 | セッションストア |
|:-----------|------------:|
| 入力～確認～完了画面間で入力情報の保持(複数タブでの画面操作を許容しない) |DBストア |
| 入力～確認～完了画面間で入力情報の保持(複数タブでの画面操作を許容する) | HIDDENストア |
| 認証情報の保持 | DBストア or HTTPストア |

###### HTTPセッションストア
サーバ上のHTTPセッションにセッション変数を保持。サーバ負荷軽減のため、軽量な情報に限る。(方式設計書)  
（例）ユーザID, 氏名, 管理者権限有無
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
