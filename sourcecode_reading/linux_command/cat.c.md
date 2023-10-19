# main()
## 前処理
### 変数・構造体定義
- 指定オプションによる処理モード変更はbool型の変数で実施
- getopt_long()で文字列オプションを受け付けるため、option型の定数配列を設定
- bindtextdomain(), textdomain()は特に何もしていない
```C
  /* Nonzero if we have ever read standard input.  */
  bool have_read_stdin = false;

  struct stat stat_buf;

  /* Variables that are set according to the specified options.  */
  bool number = false;
  bool number_nonblank = false;
  bool squeeze_blank = false;
  bool show_ends = false;
  bool show_nonprinting = false;
  bool show_tabs = false;
  int file_open_mode = O_RDONLY;

  static struct option const long_options[] =
  {
    {"number-nonblank", no_argument, nullptr, 'b'},
    {"number", no_argument, nullptr, 'n'},
    {"squeeze-blank", no_argument, nullptr, 's'},
    {"show-nonprinting", no_argument, nullptr, 'v'},
    {"show-ends", no_argument, nullptr, 'E'},
    {"show-tabs", no_argument, nullptr, 'T'},
    {"show-all", no_argument, nullptr, 'A'},
    {GETOPT_HELP_OPTION_DECL},
    {GETOPT_VERSION_OPTION_DECL},
    {nullptr, 0, nullptr, 0}
  };

  initialize_main (&argc, &argv);
  set_program_name (argv[0]);
  setlocale (LC_ALL, "");
  bindtextdomain (PACKAGE, LOCALEDIR);
  textdomain (PACKAGE);

  /* Arrange to close stdout if we exit via the
     case_GETOPT_HELP_CHAR or case_GETOPT_VERSION_CHAR code.
     Normally STDOUT_FILENO is used rather than stdout, so
     close_stdout does nothing.  */
  atexit (close_stdout);
```

### オプション解釈
- getopt_long()で単文字/文字列のオプションを解釈
- HELP/VERSIONには特別なenumで値が設定される
  - どのコマンドでも共通だから？
  - 文字列オプションで--help, --versionを用意しているので必要ない気がする
```C
  /* Parse command line options.  */

  int c;
  while ((c = getopt_long (argc, argv, "benstuvAET", long_options, nullptr))
         != -1)
    {
      switch (c)
        {
        case 'b':
          number = true;
          number_nonblank = true;
          break;

        case 'e':
          show_ends = true;
          show_nonprinting = true;
          break;

        case 'n':
          number = true;
          break;

        case 's':
          squeeze_blank = true;
          break;

        case 't':
          show_tabs = true;
          show_nonprinting = true;
          break;

        case 'u':
          /* We provide the -u feature unconditionally.  */
          break;

        case 'v':
          show_nonprinting = true;
          break;

        case 'A':
          show_nonprinting = true;
          show_ends = true;
          show_tabs = true;
          break;

        case 'E':
          show_ends = true;
          break;

        case 'T':
          show_tabs = true;
          break;

        case_GETOPT_HELP_CHAR;

        case_GETOPT_VERSION_CHAR (PROGRAM_NAME, AUTHORS);

        default:
          usage (EXIT_FAILURE);
        }
    }
```

### 標準出力のデバイス番号、i-node番号、最適化済みブロックサイズを取得
- fstat()はファイルディスクリプタで指定したファイルの情報をバッファに格納する
  - STDOUT_FILENO = 1 ( = 標準出力　※標準入力は0、標準エラー出力は2）
```C
  if (fstat (STDOUT_FILENO, &stat_buf) < 0)
    error (EXIT_FAILURE, errno, _("standard output"));

  /* Optimal size of i/o operations of output.  */
  idx_t outsize = io_blksize (&stat_buf);

  /* Device and I-node number of the output.  */
  dev_t out_dev = stat_buf.st_dev;
  ino_t out_ino = stat_buf.st_ino;
```

### 出力が通常ファイルであることの確認、ファイルオープンモードのチェック
- 行数オプション、行末表示オプション、空行圧縮オプションがいずれも無効の場合は標準出力をバイナリモードに設定する
```C
  /* True if the output is a regular file.  */
  bool out_isreg = S_ISREG (stat_buf.st_mode) != 0;

  if (! (number || show_ends || squeeze_blank))
    {
      file_open_mode |= O_BINARY;
      xset_binary_mode (STDOUT_FILENO, O_BINARY);
    }
```

## メインループ
### 変数定義
- infileは入力ファイル　※標準入力でも可
- optindは*次に解釈される*オプションのインデックス値
  - cat.cではオプションの解釈失敗（−1）までループを回すので、argindは入力ファイル（があるであろう）インデックス値を示す
```C
  infile = "-";
  int argind = optind;
  bool ok = true;
  idx_t page_size = getpagesize ();
```

### infile取得
- argindが引数の個数より少ない->argv[argind]は入力ファイル
```C
      if (argind < argc)
        infile = argv[argind];
```

### 標準入力かファイル入力かの判断
- infileは初期値がハイフンなので、↑で更新されなかった場合か*argv[argind]がハイフンだった場合*は標準入力を読み込む
  - 後者は`echo Clang | Hello - World`−＞`Hello Clang World`のように使える
  - catの本義であるところの「ファイル結合」
  - 「ディレクトリ内の全ファイルにヘッダ・フッタを結合する」といった使い方が可能
- reading_stdinの結果に応じてinput_descriptorが設定される
```C
      bool reading_stdin = STREQ (infile, "-");
      if (reading_stdin)
        {
          have_read_stdin = true;
          input_desc = STDIN_FILENO;
          if (file_open_mode & O_BINARY)
            xset_binary_mode (STDIN_FILENO, O_BINARY);
        }
      else
        {
          input_desc = open (infile, file_open_mode);
          if (input_desc < 0)
            {
              error (0, errno, "%s", quotef (infile));
              ok = false;
              continue;
            }
        }
```

### input descriptorのファイル情報を取得
- 取得できなければ、エラー
  - ok = falseて。。。どーなん
- fdadvise()はファイルへの特定のアクセス方式を宣言する
  - カーネルに適切な最適化を実行するためのアドバイス
  - SEQUENTIALならデータがシーケンシャルに（大きなオフセットの前に小さなオフセットのデータを読むように）アクセスされることを期待する
    - 「ある地点が読まれたら、次はその隣が読まれるかな」という予測が立つ場合に使う
    - ディスクIOでの高速化を狙ったものか。今どきのSSDはランダムアクセス性能が上がっているのであまり恩恵はないのかも。
  - その他のadvice
    - NORMAL（特に指定なし）
    - RANDOM（ランダムアクセス）
    - NOREUSE(データは１度しかアクセスされない）
    - WILLNEED(指定されたデータは近い将来アクセスされる）
    - DONTNEED(指定されたデータは近い将来アクセスされない）
  - [POSIX_FADVISE](https://linuxjm.osdn.jp/html/LDP_man-pages/man2/posix_fadvise.2.html)
```C
      if (fstat (input_desc, &stat_buf) < 0)
        {
          error (0, errno, "%s", quotef (infile));
          ok = false;
          goto contin;
        }

      /* Optimal size of i/o operations of input.  */
      idx_t insize = io_blksize (&stat_buf);

      fdadvise (input_desc, 0, 0, FADVISE_SEQUENTIAL);
```

### 入力ファイルと出力ファイルが同じ通常ファイルを指す場合は無駄なのでさっさとエラー扱いする
- 通常ファイルで、デバイスファイルが一致して、inodeが一致して
- lseek()はファイルの読み書きオフセットの位置を変える
  - 本文ではinput_descのオフセットを現在位置に変更して、ファイル先頭からのファイル位置をバイト数で返す
```C
      /* Don't copy a nonempty regular file to itself, as that would
         merely exhaust the output device.  It's better to catch this
         error earlier rather than later.  */

      if (out_isreg
          && stat_buf.st_dev == out_dev && stat_buf.st_ino == out_ino
          && lseek (input_desc, 0, SEEK_CUR) < stat_buf.st_size)
        {
          error (0, 0, _("%s: input file is output file"), quotef (infile));
          ok = false;
          goto contin;
        }
```

### どのバージョンのcatを使うか
- 整形が必要なオプションを指定されている場合はcat
- 整形不要で入力・出力共に通常ファイルの場合はcopy_cat
  - 単にコピーするだけなので入力バッファは空
- その他の場合はsimple_cat
- xalignalloc(): バッファ容量の確保（mallocみたいなもん？最後に呼んでるalignfreeがfree()相当か）
- 
```C
      /* Pointer to the input buffer.  */
      char *inbuf;

      /* Select which version of 'cat' to use.  If any format-oriented
         options were given use 'cat'; if not, use 'copy_cat' if it
         works, 'simple_cat' otherwise.  */

      if (! (number || show_ends || show_nonprinting
             || show_tabs || squeeze_blank))
        {
          int copy_cat_status =
            out_isreg && S_ISREG (stat_buf.st_mode) ? copy_cat () : 0;
          if (copy_cat_status != 0)
            {
              inbuf = nullptr;
              ok &= 0 < copy_cat_status;
            }
          else
            {
              insize = MAX (insize, outsize);
              inbuf = xalignalloc (page_size, insize);
              ok &= simple_cat (inbuf, insize);
            }
        }
      else
        {
          /* Allocate, with an extra byte for a newline sentinel.  */
          inbuf = xalignalloc (page_size, insize + 1);

          /* Why are
             (OUTSIZE - 1 + INSIZE * 4 + LINE_COUNTER_BUF_LEN)
             bytes allocated for the output buffer?

             A test whether output needs to be written is done when the input
             buffer empties or when a newline appears in the input.  After
             output is written, at most (OUTSIZE - 1) bytes will remain in the
             buffer.  Now INSIZE bytes of input is read.  Each input character
             may grow by a factor of 4 (by the prepending of M-^).  If all
             characters do, and no newlines appear in this block of input, we
             will have at most (OUTSIZE - 1 + INSIZE * 4) bytes in the buffer.
             If the last character in the preceding block of input was a
             newline, a line number may be written (according to the given
             options) as the first thing in the output buffer. (Done after the
             new input is read, but before processing of the input begins.)
             A line number requires seldom more than LINE_COUNTER_BUF_LEN
             positions.

             Align the output buffer to a page size boundary, for efficiency
             on some paging implementations.  */

          idx_t bufsize;
          if (ckd_mul (&bufsize, insize, 4)
              || ckd_add (&bufsize, bufsize, outsize)
              || ckd_add (&bufsize, bufsize, LINE_COUNTER_BUF_LEN - 1))
            xalloc_die ();
          char *outbuf = xalignalloc (page_size, bufsize);

          ok &= cat (inbuf, insize, outbuf, outsize, show_nonprinting,
                     show_tabs, number, number_nonblank, show_ends,
                     squeeze_blank);

          alignfree (outbuf);
        }

      alignfree (inbuf);
```
