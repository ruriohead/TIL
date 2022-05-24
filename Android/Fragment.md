## Fragment
> [フラグメント](https://developer.android.com/guide/components/fragments?hl=ja)
### フラグメントとは
- UI部品の一種
  - 1つのアクティビティに複数のフラグメントを組み合わせたマルチペインUI
  - 複数のアクティビティでのフラグメント再利用（例：タブレットとスマホのUIの作り分け）
- フラグメントは常にアクティビティでホストされている必要がある（アクティビティの上に乗っける必要がある）
  - アクティビティが一時停止したら、内部のフラグメントも一時停止
  - アクティビティが破棄されたら、内部のフラグメントも破棄
  - アクティビティ実行中のフラグメントの追加/削除/置換えは任意
- フラグメントの追加/削除/置換え（Transaction）は`FragmentTransaction`で実施する
  - add(), remove(), replace()等々のメソッドが用意されている
  - トランザクションの履歴はバックスタックで管理することもできる
    - トランザクションのコミットを実施するとバックスタック追加  
      ※ commit()を呼び出したタイミングで**それまでの変更を1つのトランザクションとして**バックスタックに追加する
    - [戻る]ボタン押下でトランザクションを戻す  
      ※ 複数変更がまとまったトランザクションは、全てが同時にもとに戻る
- アクティビティレイアウトの一部としてのフラグメントはアクティビティのビュー階層`ViewGroup`に配置される
  1. アクティビティのレイアウトファイルに`<fragment>`要素としてフラグメントを宣言
  ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:orientation="horizontal"
      android:layout_width="match_parent"
      android:layout_height="match_parent">
      <fragment android:name="com.example.news.ArticleListFragment"
              android:id="@+id/list"
              android:layout_weight="1"
              android:layout_width="0dp"
              android:layout_height="match_parent" />
      <fragment android:name="com.example.news.ArticleReaderFragment"
              android:id="@+id/viewer"
              android:layout_weight="2"
              android:layout_width="0dp"
              android:layout_height="match_parent" />
    </LinearLayout>
  ```
  
  2. アクティビティレイアウトの`ViewGroup`（やそれを継承した`<FrameLayout>`等）にコードでフラグメントを追加

  ```java
  // アクティビティのフラグメントを管理するマネージャー
  FragmentManager fragmentManager = getSupportFragmentManager();
  // 追加/削除/変更のためのトランザクションを開く
  FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

  ExampleFragment fragment = new ExampleFragment();
  // 引数は（配置先ViewGroup, 追加するフラグメント）
  fragmentTransaction.add(R.id.fragment_container, fragment);
  // バックスタックにトランザクションを追加
  fragmentTransaction.addToBackStack(null);  
  // トランザクションの適用
  fragmentTransaction.commit();
  ```

### フラグメントのライフサイクル
- `Fragment`クラスは`Activity`クラスによく似たライフサイクルを持つ
  - `onCreate()`, `onStart()`, `onPause()`, `onStop()`
  - 少なくとも以下の実装は必須
    - `onCreate()`  
      - フラグメント作成時にシステムが呼び出す  
      - フラグメントが一時停止、停止して、再開されたときに保持する必須コンポーネントの初期化を実装する
    - `onCreateView()`  
      - フラグメントが初めてUIを描画するタイミングでシステムが呼び出す  
      - このメソッドから`View`を返す必要がある（nullを返せばUIを提示しないフラグメントも実装可能）
        - レイアウトリソース（xml）からインフレートできる  
        （※ ListFragmentのサブクラスに対しては、デフォルトでListViewがonCreateView()から返されるため、実装不要）
        ```java
          public static class ExampleFragment extends Fragment {
              // LayoutInflater is given from onCreateView() itself
              @Override
              public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
                  // Inflate the layout for this fragment
                  // ・containerはフラグメントのレイアウトが（アクティビティのレイアウトから）が挿入される親ViewGroup
                  // ・falseはインフレーとされたレイアウトを2つ目のパラメータにアタッチすべきかどうかを示す
                  // ・Fragment作成時はfragmentManagerがアタッチしてくれるので不要（trueにすると親ViewGroupに同じViewを2つセットすることになる）
                  return inflater.inflate(R.layout.example_fragment, container, false);
              }
          }
        ```
    - `onPause()`  
      - ユーザーがフラグメントから離れたタイミングでシステムが呼び出す（フラグメントが破棄されようとしていない場合も含む）  
      - 現在のユーザーセッション後も維持する必要のある変更点を保存（ユーザーが戻ってこない可能性があるため）
        
### Fragmentのサブクラス
- `DialogFragment`
  - フローティングダイアログ
  - このクラスを使用してダイアログを作成するとActivityで管理されるフラグメントのバックスタックにフラグメントダイアログを組み込める
    - ユーザーは終了したフラグメントダイアログに戻ることが可能になる
- `ListFragment`
  - アイテムリストを表示する（リストの表示には`RecyclerView`の使用推奨→[RycyclerViewによるリストの作成](https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=ja)）
- `PreferenceFragmentCompat`
  - `Preference`オブジェクトの階層をリストとして表示（アプリの設定画面作成に用いる）

