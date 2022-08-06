## 数値リテラルに内のアンダースコア
[数値リテラル内のアンダースコア](https://docs.oracle.com/javase/jp/8/docs/technotes/guides/language/underscores-literals.html) 
- 桁のまとまりを分離することで可読性を高めるための仕組み
  - 要するによくある（100,000,000）のような桁数区切りをJavaの世界でも実現しようとしたもの（カンマが使えないからしゃーなしアンダースコア）

## varの型推論
[【Java】varを使うべき場合、使うべきではない場合](https://qiita.com/dhirabayashi/items/a4a13d19b41779325bb0a4a13d19b41779325bb0)
- 右辺から容易に型が分かるケースで、冗長な型宣言を減らして可読性を高めるものらしい
  ```java
  // 従来の書き方
  DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy/MM/dd");
  // varを使った書き方
  var dtf = DateTimeFormatter.ofPattern("yyyy/MM/dd");
  ```
- ローカル変数の型推論でしか使えない
- 右辺から容易に型推論できないケースでは使えない
  - var a;
  - var b = null;
  - var c = () -> {};
  - var d = {1,2,3}; => 配列の中身の型を特定できない
- 型推論のタイミングは「コンパイル時」
  - 実行中の動的型付けではない

## StringBuilder()の仕様
[StringBuilder](https://docs.oracle.com/javase/jp/8/docs/api/java/lang/StringBuilder.html)
- StringBuilder()はデフォルトで16文字分のバッファを持つ（＋16文字くらいの変更がある前提でオブジェクトを生成する）
- 文字列を引数に渡すコンストラクタでインスタンス化すると、「渡した文字列の長さ＋16文字」がバッファの大きさになる
- バッファ容量を超える文字列長になると（バッファがオーバーフローすると）自動的にバッファ容量が更新される
