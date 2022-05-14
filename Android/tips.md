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
