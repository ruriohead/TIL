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
- Stringはimmutable（不変）
→ 一度生成した文字列は変更不能
   →　一見文字列を追加・変更しているように見えて、実際はいちいち新たに文字列が生成されている
   →　ループ回数の多いfor文等に用いると、凄まじいメモリ消費量になってパフォーマンスが低下する
- StringBuilderはmutable（可変）
→　文字列の拡張に対応するバッファを持つ
   →　文字列の追加・変更は一旦バッファで受け取り、その結果を既存変数の参照先に反映する
   →　メモリ消費量が一定になり、パフォーマンス低下を防げる

> [https://qiita.com/shunsuke227ono/items/e8f34c67dcffa0fa28ad]

### arrayとArrayList
```java
// array
int[] numbers = new int[5];
String[] names = new String[5];
// ArrayList
ArrayList<Integer> myNumbers = new ArrayList<Integer>();
myNumbers.add(10);
myNumbers.add(15);
myNumbers.add(24);
myNumbers.set(2, 25);
myNumbers.remove(0);
System.out.println(myNumbers); // ->(15, 25)
```
- arrayはimmutable（不変）
→　要素の追加/削除がしたければ、新しいarrayを生成しなければいけない
- ArrayListはmutable（可変）
→　追加（add()）、削除（rmeove()）、変更（set()）、取得（get()）が自在に行える
- ArrayListの要素はオブジェクト（⇒ プリミティブ型は対応するラッバークラスを使ってオブジェクトに変換する必要がある）  
int: `Integer`, boolean: `Boolean`, char: `Character`, double: `Double`

### ArrayListとLinkedList
- 同じListインターフェースを実装しているので動作は殆ど一緒
- データの持ち方が異なる（それに起因して用途も異なる）  
1. ArrayList
- 内部にarrayを保持しているため、容量が十分でなかった場合にはより大きなarrayが生成されて古いarrayと置き換えられる（古い方は削除される）
- 使い所： 配列内の要素に対してランダムなアクセスを必要とし、要素の挿入/削除があまり必要ない場合
- 例： データベースからデータを読み込み、以降順次参照しつつ計算する場合
2. LinkedList
- データをコンテナに格納している
- LinkedListは1つ目のコンテナへのリンクを保持しており、以降の各コンテナは自身の次のコンテナへのリンクを保持している
- 使い所： 配列内の要素に対してランダムなアクセスを必要とせず、要素の挿入/削除を頻繁に行う場合
- 例： プログラム中で発生するデータの入れ物が必要な場合

![image](https://user-images.githubusercontent.com/6058309/167079292-626a4549-c7ce-4148-ab0c-5f7cf4d72183.png)  

このため以下のように処理ごとの実行速度に差がでる。  
ArrayListはデータの保管とアクセスに、LinkedListはデータの操作に向いていることがわかる  

|Description	|Operation	|ArrayList	|LinkedList | 
|:---|:---:|:---:|:---:| 
|Get an element	|get	|Fast	|Slow | 
|Set an element	|set	|Fast	|Slow | 
|Add an element (to the end of the list)	|add	|Fast	|Fast | 
|Insert an element (at an arbitrary position)	|add(i, value)	|Slow	|Fast | 
|Remove an element	|remove	|Slow	|Fast |

> [https://codegym.cc/quests/lectures/questsyntax.level08.lecture05]
> [https://qiita.com/BumpeiShimada/items/522798a380dc26c50a50]

### iterator.remove()とforループ
- イテレータはコレクション型のループに使える
- コレクション型が可変であるため、イテレータはループ中の要素を容易に変更できるように設計されている
- 例：iterator.remove()はループ中に要素の削除を実施できる（ドキュメント内の不要な改行コードをループで一括削除したりできる）  
**※これをforループ（for-eachループ）の中で実行してはいけない**  
**コードがループしようとするタイミングで、同時にコレクションのサイズを変更してしまうため繰り返し回数に矛盾が生じる**  


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
> [http://sjc-p.obx21.com/word/et/transient.html]  
- synchronized:  
メソッドが同時に一つのスレッドからのみアクセスされる（排他制御）  
プログラムにてスレッドを分けて処理しているけれど、複数スレッドで同時に処理されては困る箇所を制限  
※デッドロックに注意
> [https://qiita.com/subaru44k/items/13c52151b8d08fbc0380]
- volatile:  
要素の値がローカルスレッドにキャッシュされず、常にメインメモリから読み込まれる  
ｰ> パフォーマンス向上目的で各スレッドが変数のコピーを用意すると、メインメモリに変更があった場合に値の乖離が発生する可能性がある  
コンパイルの無駄な最適化を防ぐ用途もある（最適化の対象外になる）  
※volatileはsynchronizedの簡易版  
volatileはスレッドとメインメモリ間の変数を同期する。synchronizedは加えて排他制御も行う
> [https://qiita.com/Kohei-Sato-1221/items/8d2c08ce0b0f829e0c0a]

### Thread
- 同時に複数の処理を並行させる手法
- 複雑なタスクをバックグラウンドで実行しつつ、メインプログラムを実行したい場合に使える
- Threadの生成
  - Threadクラスを継承し、run()メソッドをオーバーライドする
  - Runnableインターフェースを実装し、run()メソッドをオーバーライドする
- Threadの実行
  - Threadクラスを継承している場合、継承先クラスのインスタンスを生成して、そのstart()メソッドを呼び出す
  ```java
  public class Main extends Thread {
      public static void main(String[] args) {
          // Threadを継承したクラスのインスタンスを生成
          // この例だと自身のインスタンスを生成していてわかりにくいな・・・
          Main thread = new Main();
          thread.start();
          System.out.println("This code is outside of the thread");
      }
      @Override
      public void run() {
          System.out.println("This code is running in a thread");
      }
  }  
  ```
  - Runnableインターフェースを実装している場合、実装しているクラスのインスタンスをThreadオブジェクトに渡して、start()メソッドを呼び出す
  ```java
  public class Main implements Runable {
      public static void main(String[] args) {
          Main obj = new Main();
          Thread thread = new Thread(obj);
          thread.start();
          System.out.println("This code is outside of the thread");
      }
      @Override
      public void run() {
          System.out.println("This code is runnninng in a thread");
      }
  }
  ```
**※メイン側とスレッド側で同一変数の読み書きをする場合に、処理の進み具合にによって値が予測不能になる可能性がある**
**この場合、`while(thread.isAlive()){}`でthreadの終了を待つなどして値を予測可能にする必要がある**

### ポリモーフィズム、継承の目的と使いどころ
- コードの再利用性を高めたいとき：新たなクラスを作成するときに、既存クラスの要素やメソッドを再利用する

### インナークラス
- Javaではクラスをネストできる（可読性やメンテナンス性を高めるために、共通して存在するクラスをグループ化する）  
- インナークラスはアウタークラスの要素やメソッドにアクセスすることができる  
- 通常のクラスと異なり、指定できるアクセス修飾子は`private`か`protected`（外部のオブジェクトからインナークラスにアクセスされたくない場合は`private`にする）
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
- `static`修飾子を指定すると、アウタークラスを生成せずにインナークラスにアクセスできる  

### 抽象クラス、抽象メソッドの用途
- セキュリティ向上：オブジェクトの特定の詳細項目を隠し、必要な詳細項目のみを表示する（インターフェースも同様）
- 開発初期に必要な機能を書き出しておく場として（インターフェースも同様）

### インターフェースと抽象化の違い
- 擬似的に他重継承を実現できる（JavaはC++のような他重継承をサポートしていない）  
インターフェースは複数実装することができる

### インターフェースのデフォルトアクセス修飾子
- メソッドは`public`,`abstract`
- 要素は`public`,`static`,`final`

### キャストのコンパイルエラーとClassCastException
- キャスト成功の可能性が全くない場合はコンパイルエラー（Generics等で方を明示することでコンパイルエラーを検知しやすくなる）
- キャスト成功の可能性はあって、実行したらエラーになったものがClassCastException
  ```java
  // コンパイル時点ではObject -> Stringの変換は正しいものとされる
  // 但し実行すると、iはInteger型であり、String型にキャストできない
  Object i = Integer.valueOf(42); 
  String s = (String)i; // ClassCastException thrown here.
  ```
> https://stackoverflow.com/questions/907360/explanation-of-classcastexception-in-java

### 匿名クラスとラムダ式で扱える変数について
- クラス変数やstatic変数は扱える（参照、値の変更）
- finalでないローカル変数や引数は扱えない（参照は可能、値の変更不可）  
  ```java
  public class SampleClass {
      private int classField = 0;
      private static int staticField = 0;
      private void process() {
          DoSomethingInterface functionalInterface = () -> {
              classField ++;  // ← この変数を扱うことは可能。
              staticField ++;  // ← この変数を扱うことは可能。
          };
          System.out.println("Before; classField =" + classField);
          System.out.println("Before; staticField =" + staticField);
          functionalInterface.doSomething(); // 処理を実行
          System.out.println("After; classField =" + classField);
          System.out.println("After; staticField =" + staticField);
      }
      @FunctionalInterface
      public interface DoSomethingInterface {
          void doSomething();
      }
      public static void main(String[] args) {
          SampleClass sample = new SampleClass();
          sample.process();
      }
  }
  ```
  ```java
  private void process() {
      final int i = 0;
      int j = 0;
      DoSomethingInterface finctionInterface = () -> {
          i++; // <- OK
          System.out.println("j=" + j); // 参照しているだけなのでOK
          j++; // <- この変数は扱えないのでエラー
      }
  }
  ```
- ローカル変数や引数が何らかのクラスだった場合、そのクラスが持つクラス変数の値を変更可能
- クラス自体を変更することは不可能（参照は可能）  
  ```java
  public class DummyClass {
      public int field;
  }
  
  private void process(DummyClass argClass) {
      DummyClass localFiledClass = new DummyClass();
      DoSomethingInterface functionalInterface = () -> {
      //        localFiledClass = new DummyClass(); // ← 不可
      //        argClass = new DummyClass(); // ← 不可
          localFiledClass.field++; // ← 可能
          argClass.field++; // ← 可能
  ……
  ```
  
