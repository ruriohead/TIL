### this
1. 自分自身を指定
- メンバ変数名とローカル変数名が重複した際のスコープ解決
- this.variable = variableのように接頭辞としてthisをつけることでメンバ変数であることを明示する
2. 自クラスの別のコンストラクタを呼び出す
- コンストラクタが複数存在する場合に、あるコンストラクタから別のコンストラクタを明示的に呼び出すことができる
- 重複している処理を省いたりするのに使う

  **※これを継承元について実施しているのがsuper()**

3. 他オブジェクトに自身の参照を渡す
- 別のクラスのコンストラクタに自分自身の参照情報を代入する
- 受け取った側はこの情報をクラス内の変数に格納するなどして利用する
- オブジェクト間の関連付け
```java
class Person {
   public String firstName;
   public String lastName;
   public int age;
   private double height;
   private double weight;
   
   // 生成する別クラスのオブジェクトに自身の参照情報を代入
   // ※thisで代入されていてもprivate修飾子の要素にはアクセスできない（まぁ当たり前か）
   public void function() {
      makeSelfIntroduction si =  new makeSelfIntroduction(this);
   }
}

class makeSelfIntroduction {
   // コンストラクタ引数で受け取ったPersonクラスの変数値を利用   
   makeSelfIntroduction(Person p) {
      System.out.println("my name is " + p.firstName + " " + p.lastName + ".");
      System.out.println("I'm " + p.age + " years old.");
   }
}
```

### StringとStringBuilder
- Stringはimmutable(不変）
→ 一度生成した文字列は変更不能
   →　一見文字列を追加・変更しているように見えて、実際はいちいち新たに文字列が生成されている
   →　ループ回数の多いfor文等に用いると、凄まじいメモリ消費量になってパフォーマンスが低下する
- StringBuilderはmutable(可変）
→　文字列の拡張に対応するバッファを持つ
   →　文字列の追加・変更は一旦バッファで受け取り、その結果を既存変数の参照先に反映する
   →　メモリ消費量が一定になり、パフォーマンス低下を防げる

参考：https://qiita.com/shunsuke227ono/items/e8f34c67dcffa0fa28ad
