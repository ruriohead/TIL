### プリミティブ型と参照型の違い
- プリミティブ型はJavaで定義済。参照型はString型を除き、プログラマによって定義される
- 参照型は特定の操作を実施するメソッドを呼び出すのに利用できるが、プリミティブ型はできない
- プリミティブ型は常に値を持つが、参照型は`null`を持ちうる
- プリミティブ型はlowercase（int, boolean, double...)。参照型はuppercase（Stringやユーザー定義クラス）
- プリミティブ型のサイズはデータ型に依存するが、参照型はすべて同じサイズ

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

参考[https://qiita.com/shunsuke227ono/items/e8f34c67dcffa0fa28ad]

### 修飾子
1. アクセス修飾子
- public:    すべてのクラスからアクセス可能
- private:   宣言されたクラスの内部でのみアクセス可能
- default:   同一パッケージ内からアクセス可能（アクセス修飾子のないクラス、要素、メソッド、コンストラクタは自動でこれになる）
- protected: 同一パッケージとサブクラスからアクセス可能
2. 非アクセス修飾子
- final:         継承不可
- static:        要素やメソッドがオブジェクトではなく、クラスに属している
- abstract:  
（クラスなら）オブジェクトの生成に利用できない（継承される必要がある）  
（メソッドなら）抽象クラスでのみ利用可能。中身は継承したサブクラスで提供される（抽象メソッドはアイデアを記載するのみ）
- transient:  
シリアライズ（直列化）による情報のストリーム化の対象外になる  
ストリーム化： オブジェクトをファイルとして扱ったり、ネットワーク上でやり取りできるようにするための方法  
※ staticなフィールドはそもそもクラスに属するのでオブジェクトのストリーム化の対象外  
参考[http://sjc-p.obx21.com/word/et/transient.html]  
- synchronized:  
メソッドが同時に一つのスレッドからのみアクセスされる（排他制御）  
プログラムにてスレッドを分けて処理しているけれど、複数スレッドで同時に処理されては困る箇所を制限  
※デッドロックに注意
参考[https://qiita.com/subaru44k/items/13c52151b8d08fbc0380]
- volatile:  
要素の値がローカルスレッドにキャッシュされず、常にメインメモリから読み込まれる  
ｰ> パフォーマンス向上目的で各スレッドが変数のコピーを用意すると、メインメモリに変更があった場合に値の乖離が発生する可能性がある  
コンパイルの無駄な最適化を防ぐ用途もある（最適化の対象外になる）  
※volatileはsynchronizedの簡易版  
volatileはスレッドとメインメモリ間の変数を同期する。synchronizedは加えて排他制御も行う
参考[https://qiita.com/Kohei-Sato-1221/items/8d2c08ce0b0f829e0c0a]

### ポリモーフィズム、継承の目的と使いどころ
- コードの再利用性を高めたいとき：新たなクラスを作成するときに、既存クラスの要素やメソッドを再利用する

### インナークラス
Javaではクラスをネストできる（可読性やメンテナンス性を高めるために、共通して存在するクラスをグループ化する）  
インナークラスはアウタークラスの要素やメソッドにアクセスすることができる  
通常のクラスと異なり、指定できるアクセス修飾子は`private`か`protected`（外部のオブジェクトからインナークラスにアクセスされたくない場合は`private`にする）
```java
例
class OuterClass {
  int x = 10;

  private class InnerClass {
    int y = 5;
  }
}

public class Main {
  public static void main(String[] args) {
    OuterClass myOuter = new OuterClass();
    // クラスの外側（メインクラス）からインナークラスにアクセス
    OuterClass.InnerClass myInner = myOuter.new InnerClass();
    System.out.println(myInner.y + myOuter.x);
  }
}
```
はエラーになる  
`static`修飾子を指定すると、アウタークラスを生成せずにインナークラスにアクセスできる  
