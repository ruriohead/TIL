# Room
> [Room を使用してローカル データベースにデータを保存する](https://developer.android.com/training/data-storage/room?hl=ja)
## Roomとは
- SQLite全体に抽象化レイヤを提供する永続ライブラリ（永続＝ライフサイクルに無関係）
- Google推奨なので、特に理由がなければSQLiteを直接使用するのは避ける
  - Javaのドキュメントがあまり充実していないのが玉に瑕（コード例がKotlinしかないページが多い）

## セットアップ
- `build.gradle`の`dependencies`に以下を追記
```java
dependencies {
    def room_version = "2.4.2"

    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version"

    // optional - RxJava2 support for Room
    implementation "androidx.room:room-rxjava2:$room_version"

    // optional - RxJava3 support for Room
    implementation "androidx.room:room-rxjava3:$room_version"

    // optional - Guava support for Room, including Optional and ListenableFuture
    implementation "androidx.room:room-guava:$room_version"

    // optional - Test helpers
    testImplementation "androidx.room:room-testing:$room_version"

    // optional - Paging 3 Integration
    implementation "androidx.room:room-paging:2.5.0-alpha01"
}
```

## 主要コンポーネント
- Room Databaseクラス
  - DBを保持し、アプリの永続データに対する基礎的な接続のメインアクセスポイントとして機能する
  - 以下のように、関連するデータエンティティと、DAOについて記載し、`RoomDatabase`を継承した抽象クラスを作成する
  ```java
  @Database(entities = {User.class}, version = 1)
  public abstract class AppDatabase extends RoomDatabase {
      public abstract UserDao userDao();
  }
  ```
- Data Access Object (DAO)  
> [Room DAO を使用してデータにアクセスする](https://developer.android.com/training/data-storage/room/accessing-data?hl=ja) 
  - アプリがDBのデータのクエリ/更新/挿入/削除に使用できるメソッドを提供するオブジェクト
  - @Daoアノテーションをつけたインターフェースか抽象クラスで定義する
  - コンビニエンスメソッドはSQL文の記述なしにシンプルな挿入/更新/削除操作を定義する
    - `@Insert`
    - `@Update`
    - `@Delete`
  - クエリメソッドはSQL文を記述して、データのクエリやより複雑な操作を定義する
    - `@Query("SQL statement")`
  ```java
  @Dao
  public interface UserDao {
      @Query("SELECT * FROM user")
      List<User> getAll();

      @Query("SELECT * FROM user WHERE uid IN (:userIds)")
      List<User> loadAllByIds(int[] userIds);

      @Query("SELECT * FROM user WHERE first_name LIKE :first AND " +
             "last_name LIKE :last LIMIT 1")
      User findByName(String first, String last);

      @Insert
      void insertAll(User... users);

      @Delete
      void delete(User user);
  }
  ```
- Data Entities  
> [Room エンティティを使用してデータを定義する](https://developer.android.com/training/data-storage/room/defining-data?hl=ja)
  - DBに保存するオブジェクトを表すエンティティの定義（＝データベーススキーマ）
  - 以下のように主キーや列情報を含むクラスを定義する
  ```java
  @Entity
  public class User {
      @PrimaryKey
      public int uid;

      @ColumnInfo(name = "first_name")
      public String firstName;

      @ColumnInfo(name = "last_name")
      public String lastName;
  }
  ```
  ![image](https://user-images.githubusercontent.com/6058309/170301265-aa16e609-d462-4f22-af05-1c4ad6aad169.png)
  
- 上記3つを定義したらデータべース・DAOのインスタンスを作成してデータベースを操作できる
```java
AppDatabase db = Room.databaseBuilder(getApplicationContext(),
                                      AppDatabase.class, 
                                      "database-name").build();
        
UserDao userDao = db.userDao();
List<User> users = userDao.getAll();
```
## 実際の開発プロセス(RoomDatabaseを使ったMVVM開発）
- Room Databaseを使ってMVVMアーキテクチャで開発をするために、下図のようなコンポーネントに分けて実装を進める
> [【Android】分かった気になれる！アーキテクチャ・MVVM概説](https://qiita.com/iTakahiro/items/6b1b22efa69e55cea3fa)  
> [【Android】はじめてのRoom](https://qiita.com/iTakahiro/items/7e0d63140ae4dac10d18#%E3%81%AF%E3%81%98%E3%82%81%E3%81%AB)  

![image](https://user-images.githubusercontent.com/6058309/171435684-f4e7d4d0-a9ce-4d25-8d57-a1b1b8a6b76f.png)

