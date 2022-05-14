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
