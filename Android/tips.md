# 開発していて気づいたことたち

## Android Studio関連
### メソッド名にカーソル合わせてF4キーを押すと、その関数の定義箇所に飛べる
真っ先にAndroid Developersでリファレンス探すのもいいけど、オフラインでさっと情報を確認するならこっちのほうが速い

### メモリリークについて
- Android10以降は以前に比べてメモリリークしにくくなっている様子
  > [ART ガベージ コレクションのデバッグ](https://source.android.google.cn/devices/tech/dalvik/gc-debug?hl=ja)
- 開発中のアプリがメモリリークしていないかを調べるツールは沢山あるので導入すると良い
  1. [Memory Profiler を使用してアプリのメモリ使用量を調べる](https://developer.android.com/studio/profile/memory-profiler?hl=ja)
  2. [LeakCanary](https://square.github.io/leakcanary/)

### API32時点でエミュレータのモバイル回線を設定でOFFにすることはできないらしい（バグ）
- 迂回策として機内モードを使用するとよい
- 2010年時点でバグレポート出てるのに...
  > [How to simulate total network loss in Android Emulator](https://stackoverflow.com/questions/4154815/how-to-simulate-total-network-loss-in-android-emulator?rq=1)

## メソッド関連
### findViewById()とその後の操作は分けなきゃいけないっぽい
ViewでgetText()メソッドが解決できないとある   

[View.findViewById()のリファレンス](https://developer.android.com/reference/android/app/Activity#findViewById(int))に書いてあったわ
> Note: In most cases -- depending on compiler support -- the resulting view is automatically cast to the target class type.
> If the target class type is unconstrained, an explicit cast may be necessary.
> 取得結果は通常適切なターゲットクラスにキャストされるけど、ターゲットクラスに制約がない場合は明示的にキャストする必要がある

今回の場合は明示的なキャストが実施されないまま（View型のまま）getText()メソッドを呼び出すことになるのがNGぽい  
`(TextView)findViewById(R.id.text).getText()`みたいな書き方も考えたけど、どうもキャストより先にメソッド探しに行く様子（そんなもん？）  
大人しく宣言と操作を分けよう（これは他の変数についても一緒なのかも）
```java
// OK

TextView textView = findViewById(R.id.text);
String text = textView.getText();

// NG

// Cannot resolve method 'getText' in 'View'
String textView = findViewById(R.id.text).getText();

```

### findViewById()の使用条件

[View.findViewById()のリファレンス](https://developer.android.com/reference/android/app/Activity#findViewById(int))を読んでてもう一つ気づいた
> Finds a view that was identified by the android:id XML attribute __that was processed in onCreate(Bundle)__.  
> findViewById()の対象は __onCreate(Bundle)で処理された__ android:idアトリビュート
> 
だからサンプルアプリでは、いつも初手`onCreate()`内で`findViewById()`って書き方になってたのね

### ボタンを連打されてもonClickListenerが一度しか走らないようにする
> [Android OnClickListener Prevent multiple clicks](https://gist.github.com/hilfritz/5a8ca9e172918bc224f03c3dac9c39f3)
- View.OnClickListenerを実装した抽象クラスで、一定期間の再クリック防止機能を持たせる

### LiveDataはライフサイクルに応じた監視が可能なデータホルダークラス
> [LiveData の概要](https://developer.android.com/topic/libraries/architecture/livedata.html?hl=ja)
- Activity/Fragment/Serviceのライフサイクルを考慮して監視される
  - アクティブなアプリコンポーネントでの監視のみ更新する
    - 停止されたアクティビティに起因するクラッシュが発生しない
  - 関連付けられたライフサイクルが破棄されたときに自身をクリーンアップするのでメモリリークしない
  - 非アクティブ化後に再度アクティブになったときに、最新のデータを受け取る（例外的な動作）
  - Actiity/Fragmentの再作成（デバイスの回転など）後に最新のデータをすぐに受け取る
- LiveDataは通常`onCreate()`で監視を開始するのが適当
  - Activity/Fragmentの`onResume()`メソッドからの冗長な呼び出しをシステムが行わないようにできる
- Roomと併用するとバックグラウンドでのDBの非同期更新が完了次第、UIを更新できる

### LayoutInflater.inflateのattachToRoot
> [LayoutInflater.inflateのattachToRootとは何か？](https://qiita.com/nemo-855/items/e8521fed9392be72b3d9)  
- `inflate(resource: Int, root: ViewGroup?, attachToRoot: Boolean): View`の第一引数はViewに変換したいlayoutのxml, 第二引数は生成するViewをセットする親ViewGroup, 第三引数は第二引数のViewGroupへのViewのセットを自動でするか手動でするか　　

  > 僕がこのLayoutInflaterを記述していたのは大抵FragmentのonCreateViewやRecyclerViewのAdapterのonCreateViewHolder等でした。
  > これらのようなタイミングではLayoutInflater.inflateでViewを作成するタイミングでさらにそれを親のViewGroupにセットする必要がありません。
  > なぜならFragmentを作成するときはfragmentManager、RecyclerViewの場合はAdapterが代わりにその役割を果たしてくれているからです。
  > なのでonCreateViewやonCreateViewHolderの内部でLayoutInflater.inflateをする時にattachToRootをtrueにしてしまうと、
  > 親ViewGroupに同じViewを二つセットすることになってしまいIllegalStateExceptionが起きてしまいます。
  > なのでfalseをセットしないといけないらしいです。

### 

## レイアウト関連
### タイトルバー無効化
1. アプリ全体
`res/values/themes`の3行目  
`<style name="Theme.StudyApp" parent="Theme.MaterialComponents.DayNight.DarkActionBar">`を  
`<style name="Theme.StudyApp" parent="Theme.MaterialComponents.DayNight.NoActionBar">`に変更

2. アクティビティ毎に設定
`onCreate`に以下を追加  
```java
ActionBar actionBar = getSupportActionBar();
    if (actionBar != null) {
        actionBar.hide();
    }
```
### Viewのセンタライズ
- LinearLayoutなら`android:gravity="center"`に設定
- RelativeLayoutなら`android:layout_centerVertical="true"`に設定

### ボタン（やビューの等間隔配置）
`layout_width="match_parent"`(or `layout_width="0dp"`)と`layout_weight="1"`を設定  
`layout_weight`は並んでいるオブジェクトの総和に対する表示比率（→1:1:1とすれば、総和3に対して等間隔）
```java
        <LinearLayout
            android:orientation="horizontal"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <Button
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="Btn1"/>
            <Button
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="Btn2"/>
            <Button
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_weight="1"
             android:text="Btn3"/>
        </LinearLayout>
```

### テキストサイズを画面サイズ依存で自動調整する
以下を使う（※APIレベル26以上でのみ使用可能（最小APIレベルがそれを下回るとワーニング出る））
```java
    android:autoSizeTextType="uniform"
    android:autoSizeMinTextSize="最小文字サイズ"
    android:autoSizeMaxTextSize="最大文字サイズ"
```

### 角丸のViewの作り方
1. drawableにcornerで`android:radius`アトリビュートを持つレイアウトを作成  
`android:radius`は四角全て。それぞれの角は`android:bottomRight/LeftRadius`, `android:topRight/LeftRadius`で個別設定も可能
```xml
例
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape android:shape="rectangle">
            <corners
                android:radius="20dp"/>
            <solid
                android:color="@color/purple_500"/>
        </shape>
    </item>
</selector>
```
2. Viewの`android:background`アトリビュートで`@drawable/作成したレイアウト名`で指定する
※ Dialogに角丸を適用するには上記に加えて、Dialogの呼び出し元のView（Dialogを仮置きしている領域？）を透過にする必要がある  
　「Dialog自体を角丸に設定しても、呼び出し元では領域が矩形のまま→重ねてみたら矩形で表示される」という理屈らしい  
　・・・なんかsetBackgroundDrawable()は非推奨っぽい？（要確認）
> [How to make custom dialog with rounded corners in android](https://stackoverflow.com/questions/28937106/how-to-make-custom-dialog-with-rounded-corners-in-android)
```java
    @NonNull
    @Override
    public Dialog onCreateDialog(@Nullable Bundle savedInstanceState) {
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());

        // カスタムレイアウトの生成
        if (getActivity() != null) {
            popupView = getActivity().getLayoutInflater().inflate(R.layout.popup_menu, null);
        }

        Button closePopUpButton = popupView.findViewById(R.id.dialog_inner_close_button);
        closePopUpButton.setOnClickListener(v -> {
            Log.d("dialog button touched", "close dialog");
            getDialog().dismiss();
        });

        // ViewをpopUpDialog.Builderに追加
        builder.setView(popupView);

        // Dialog生成
        popUpDialog = builder.create();
        // Set the background of the dialog's root view to transparent,
        // because Android puts your dialog layout within a root view that hides the corners in your custom layout.
        popUpDialog.getWindow().setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
        popUpDialog.show();

        return popUpDialog;
    }
```

### Fragment上にSnackBarを表示する
```java
Snackbar snackBar = Snackbar.make(getActivity().findViewById(android.R.id.content),
           "Look at me, I'm a fancy snackbar", Snackbar.LENGTH_LONG);
snackBar.show();
```

### material buttonをボタンに表示する
[drawable start not working with material button](https://stackoverflow.com/questions/61353353/drawable-start-not-working-with-material-button)  
[Buttons](https://material.io/components/buttons/android#text-button)
- ボタンにテキスト付きでアイコンを表示したい
  - 通常の画像を使用する場合、`android:drawableStart`,`android:drawableEnd`,`android:drawableTop`,`android:drawableBottom`等を使う
  - material buttonを使用する場合は上記が使えないので、代わりに`app:icon="@drawable/filename"`を使う
    - `app:iconGravity`や`app:iconTint`で位置や色も自在

### logcatのファイル出力
```java
Process process = Runtime.getRuntime().exec(new String[] {"logcat", "-d", "-f", <file>});
```
-dでログを画面にダンプ  
-fでダンプ先をコンソールからファイルに変更  
process.waitfor()で大きいサイズのログがキチンと吐き出されることを担保したり、process.destroy()で処理完了後のプロセスを停止したりするとよい
