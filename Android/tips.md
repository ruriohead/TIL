# 開発していて気づいたことたち

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

