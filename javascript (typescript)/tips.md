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
