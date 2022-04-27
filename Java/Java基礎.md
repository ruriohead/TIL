# StringとStringBuilder
- Stringはimmutable(不変）
→ 一度生成した文字列は変更不能
   →　一見文字列を追加・変更しているように見えて、実際はいちいち新たに文字列が生成されている
   →　ループ回数の多いfor文等に用いると、凄まじいメモリ消費量になってパフォーマンスが低下する
- StringBuilderはmutable(可変）
→　文字列の拡張に対応するバッファを持つ
   →　文字列の追加・変更は一旦バッファで受け取り、その結果を既存変数の参照先に反映する
   →　メモリ消費量が一定になり、パフォーマンス低下を防げる
参考：https://qiita.com/shunsuke227ono/items/e8f34c67dcffa0fa28ad
