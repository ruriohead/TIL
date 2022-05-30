## 学んだこと
### map
> [Array.prototype.map()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
- 与えられた関数を配列のすべての要素に対して呼び出して、その結果から新しい配列を生成する
  - 配列から要素の一部を取り出す
  ```javascript
  const arrayObj = [
    {
      item1: "1",
      item2: "2",
      item3: "3",
      item4: "4"
    },
    {
      item1: "5",
      item2: "6",
      item3: "7",
      item4: "8"
    }
    ];
    
  const map1 = arrayObj.map(data => [data.item1, data.item2]);
  console.log(map1);
  // expected output: > Array [Array ["1", "2"], Array ["5", "6"]]
  
  const map2 = arrayObj.map(data => data.item3);
  console.log(map2);
  // expected output: > Array ["3", "7"]
  ```

### 大量の非同期処理を捌く
> [Promise.all](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)  
> [大量のPromiseを捌く手段](https://zenn.dev/e_chan1007/books/822529ef2f981d/viewer/b3099e)
- プロミスの集合を入力として、入力したプロミスの結果の配列に解決される、単一のPromiseを返す
  - 入力したプロミスが全て解決されるか、入力にプロミスが含まれていない場合に解決される
  - 入力したプロミスのいずれかが拒否されるとエラー発生
  ```javascript
  const promise1 = Promise.resolve(3);
  const promise2 = 42;
  const promise3 = new Promise((resolve, reject) => {
    setTimeout(resolve, 100, 'foo');
  });

  Promise.all([promise1, promise2, promise3]).then((values) => {
    console.log(values);
  });
  // expected output: Array [3, 42, "foo"]
  ```
  
- forEachでは処理できない
  > [Using async/await with a forEach loop](https://stackoverflow.com/questions/37576685/using-async-await-with-a-foreach-loop/37576787#37576787)
  - forEachのコールバック関数の中ではawaitされるが、コールバック関数の実行そのものはawaitされないため処理が未完のままループを抜けてしまう
