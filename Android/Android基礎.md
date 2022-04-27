### setContentView
1. setContentView(int layoutResID)
- レイアアウトXMLファイルのIDを指定```例：R.layout.activity_main```
- xmlの設定は静的(static)なのでアプリ起動中に変更することは簡単にはできない
2. setContentView(View view)
- レイアウトを動的に変化させたい場合に用いる
3. setContentView(View view, ViewGroup.LayoutParams params)
