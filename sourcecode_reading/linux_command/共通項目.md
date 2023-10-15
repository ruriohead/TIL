### fputsとprintfの使い分け
- フォーマットが不要な場合標準出力宛にfputs
  ```C
  fputs (HELP_OPTION_DESCRIPTION, stdout);
  ```
- フォーマットが必要な場合printf
  ```C
        printf (_("\Usage: %s [OPTION]... [FILE]...\n\"), program_name);
  ```
### 処理が複数行になるマクロはdo-while(0)で括る
- マクロが展開された際の意図しない副作用を防ぐためのプラクティス
- while(0)なので結局一回しか実行できないが、展開する処理が前後の処理から独立するメリットがある
- do-while(0)で括らないと以下のような副作用がある
  - 行末の；（セミコロン）を強制できない
  - 記述する場所によっては意図しない挙動になる
  - ビルドレベルによって記述できる箇所が変わる

    [DO-WHILE(0)によるマクロラッピング](https://cpp.aquariuscode.com/do-while-macro)
- 今どきはそもそもマクロを使用せず、inline関数を定義した方がいい

  [inline](https://kaworu.jpn.org/c/inline)

### getopt_long()
- ハイフンと単文字で表現したオプションだと段々わかりにくくなってきたので、--versionのように文字列でオプションを表現するようになった
- その解釈に使用するのがgetopt_long()
  - getopt_long()だと渡されたオプションを単文字としても文字列としても解釈しようとする
  ```C
       int getopt_long(int argc, char * const argv[],
                  const char *optstring,
                  const struct option *longopts, int *longindex);
  ```
  - optstringには単文字オプションをまとめて引数に渡す（"abcde"のように）
  - longoptsにはoption構造体の配列をまとめて引数に渡す
    ```C
    struct option {
      const char *name;
      int has_arg;
      int *flag;
      int val;
    };
    ```
- 文字列オプションを優先的に解釈させるgetopt_long_only()も存在する
- [コマンドラインオプションの処理](https://www.mm2d.net/main/prog/c/getopt-03.html)
