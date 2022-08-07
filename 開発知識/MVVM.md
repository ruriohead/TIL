# MVVM (Model-View-ViewModel)
## MVVMとは
[AndroidでMVVM](https://tech.nri-net.com/entry/mvvm_in_android)
- グーグル推奨のアーキテクチャパターン  
![image](https://user-images.githubusercontent.com/6058309/171560570-4d562bab-ab5c-42f3-a46d-171fa08e27ec.png)


- 開発者視点
  - コンポーネントの役割を明確化
    - データの管理が楽になる
    - テストしやすくなる
    - コードの書き換え工数の削減
    - ケアレスミスの削減
- ユーザー視点
  - データをActivityやFragmentのライフサイクルとは無関係な永続化モデルとして保存
  - UIの操作はモデルで実施
    - アプリUIをより早く表示させられる（軽量なUIにデータをモデルから挿入する）
    - メモリ不足でアプリが落ちる可能性が低くなる（ActivityやFragmentに高負荷な処理を積み上げる必要がなくなる）

## コンポーネント
- 基本ルール：　各コンポーネントは矢印の方向にしか参照できない

### View
> [LiveDataの概要](https://developer.android.com/topic/libraries/architecture/livedata.html?hl=ja)  
- UIの表示・更新
  - ViewModelを介してデータを取得し、UIに表示する
  - ViewModelにLiveDta（監視可能なデータホルダークラス）のobserverを置いて変更を検知することで、随時UIを更新する
- ユーザー操作の検知
  - （例）ボタン押下を検知して画面遷移
  - （例）入力フォームへの入力完了を検知して、ViewModelにデータの更新依頼を出す（≠ **UI自体がデータ更新**）

### ViewModel
- ViewがUIを表示するために必要なデータをRepositoryを介して取得し、整形してからViewに渡す
- Viewから送られてきた値をRepositoryを介して保存するためのメソッドを呼ぶ

### (Repository)
- データの取得方法・保存方法の把握
  - データをどこ（どのModel）から取得するか
  - どういう条件で取得するか
- データ取得・保存がしやすいようなAPIを提供
- DAOパターンより高い抽象度で、エンティティの操作から現実の永続化ストレージを完全に隠蔽する
  - データベースに直接アクセスするインスタンスを作らせない
  - Repositoryのユーザ（ViewModel）は永続化ストレージが何であるか（MySQL? Redis?）を意識せずに保存や検索ができる
  - Modelからのデータ取得をRepositoryに委任すると、Repositoryがデータの取得先を変更してもViewModelを変更する必要がなくなる  
    - ViewModelはRepositoryにデータ取得/更新/保存を要求する方法しかしらないため
- ViewModelに含める場合もあるが、分けて設置することでViewModelの負担を減らせる  
  

### Model
- データを実体として保存している（永続化）
- Room等
