## Fragment
> [フラグメント](https://developer.android.com/guide/components/fragments?hl=ja)
> [今さら聞けない Activity と Fragment の使い分け](https://qiita.com/KeithYokoma/items/c41b22bda8c8d924d8cd)
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
      - このメソッドから`View`を返す必要がある
        - nullを返せばUIを提示しないフラグメントも実装可能
          - Activityのためのデータを、ライフサイクルに合わせて管理するホルダーのような役割を持たせられる
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
  - その他のライフサイクル
    - onAttach()
      - フラグメントがアクティビティと関連づけられたときに呼び出される
      - ここで`Activity`が渡される
    - onActivityCreated()
      - アクティビティのonCreated()メソッドから戻ったときに呼び出される
    - onDestroyView()
      - フラグメントに関連付けられたビュー階層が削除されたときに呼び出される
    - onDetach()
      - フラグメントとアクティビティの関連付けが解除されたときに呼び出される  
    
    ![image](https://user-images.githubusercontent.com/6058309/169958924-a98f9de4-d282-44f1-9b2a-560b7b1f055a.png)

        
### Fragmentのサブクラス
- `DialogFragment`
  - フローティングダイアログ
  - このクラスを使用してダイアログを作成するとActivityで管理されるフラグメントのバックスタックにフラグメントダイアログを組み込める
    - ユーザーは終了したフラグメントダイアログに戻ることが可能になる
- `ListFragment`
  - アイテムリストを表示する（リストの表示には`RecyclerView`の使用推奨→[RycyclerViewによるリストの作成](https://developer.android.com/guide/topics/ui/layout/recyclerview?hl=ja)）
- `PreferenceFragmentCompat`
  - `Preference`オブジェクトの階層をリストとして表示（アプリの設定画面作成に用いる）

### アクティビティと通信する
- `Fragment`は`FragmentActivity`から独立したオブジェクトとして実装されている（使い回せる）が、フラグメントのインスタンスはホストされているアクティビティに直接結びついている
- フラグメント側からは`getActivity()`を使用して`FragmentActivity`インスタンスにアクセスでき、アクティビティのレイアウトでビューを見つけたりできる
```java
View listView = getActivity().findViewById(R.id.list);
```

- アクティビティ側からは`findFragmentById()`や`findFragmentByTag()`を使って`FragmentManager`から`Fragment`への参照を取得できる（これによりフラグメント内のメソッドを呼び出すこともできる）
```java
ExampleFragment fragment = (ExampleFragment) getSupportFragmentManager().findFragmentById(R.id.example_fragment);
```

- アクティビティへのイベントコールバックを作成する
  -  [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ja)では処理できないイベントをアクティビティやアクティビティがホストする他のフラグメントに伝える必要がある場合、フラグメント内にコールバックインターフェースを定義できる（＝ホストでの実装を強制）
  > [FragmentからActivityにコールバックする方法2017](https://qiita.com/Nkzn/items/fca698f31d3c9c335a80)
  ```java
  public static class FragmentA extends ListFragment {
    OnArticleSelectedListener listener;
    ...
    // Container Activity must implement this interface
    // ex. public class MainActivity extends Activity implements FragmentA.OnArticleSelectedListener {}
    public interface OnArticleSelectedListener {
        public void onArticleSelected(Uri articleUri);
    }
    ...

    // ホストアクティビティがOnArticleSelectedListenerを実装していないと例外発生
    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        try {
            listener = (OnArticleSelectedListener) context;
        } catch (ClassCastException e) {
            throw new ClassCastException(context.toString() + " must implement OnArticleSelectedListener");
        }
    }
    ...
    
    // フラグメントのメソッドからホストアクティビティに情報を伝える
    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Append the clicked item's row ID with the content provider Uri
        Uri noteUri = ContentUris.withAppendedId(ArticleColumns.CONTENT_URI, id);
        // Send the event and Uri to the host activity
        listener.onArticleSelected(noteUri);
    }
  }
  ```
