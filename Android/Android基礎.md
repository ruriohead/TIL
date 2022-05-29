### setContentView
1. setContentView(int layoutResID)
- レイアアウトXMLファイルのIDを指定```例：R.layout.activity_main```
- xmlの設定は静的(static)なのでアプリ起動中に変更することは簡単にはできない
2. setContentView(View view)
- レイアウトを動的に変化させたい場合に用いる
3. setContentView(View view, ViewGroup.LayoutParams params)

### Context
> [Androidの勉強：Contextについて](https://qiita.com/iduchikun/items/34b3ae26cfc438e7e5dc)  
> [【Android】Context って何？？](https://qiita.com/tkmd35/items/e6abeed6ac68cbac09ed)

- 文字通り「文脈」を表す
  - アプリのどのアクティビティに紐付いた操作であるか
  - アプリケーション全体に紐付いた操作であるか
- Contextの取得方法は
  - Activityのthis (= Activityが実体）
  - ActivityやApplicationの`getApplicationContext` （= Applicationが実体）
  - **ViewやFragmentのgetContext (= Activityが実体)**
    - 「呼び出し元は何か？」を問われている
- Activity ContextとApplication Contextの使い分け
  - Activityのライフサイクル内でContextのライフサイクルが終了するならばActivity Context
  - Activityのライフサイクルを超える範囲でContextを渡すのであればApplication Contextの使用を検討
    - Activity自体が終了しているのにContextへの参照が残っているとGCができない（⇒メモリリークを引き起こす）
