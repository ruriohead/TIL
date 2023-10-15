### fputsとprintfの使い分け
- フォーマットが不要な場合標準出力宛にfputs
  ```C
  fputs (HELP_OPTION_DESCRIPTION, stdout);
  ```
- フォーマットが必要な場合printf
  ```C
        printf (_("\Usage: %s [OPTION]... [FILE]...\n\"), program_name);
  ```
