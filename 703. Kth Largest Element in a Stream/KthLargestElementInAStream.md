# step1 何も見ずに解く
- PriorityQueueのカテゴリなので一旦それを使う前提で考える
  - PriorityQueueを使用したことがないのでリファレンスを参照する
    - メソッド名に馴染みが薄い
      - `offer`: 要素を追加
      - `poll`: 最優先要素を取り出して削除
    - 内部的には配列を使用した完全二分木で実装されている
      - Priorityがn番目に高い要素を取り出そうと思ったらそこまで`poll()`しないと取り出せなさそう
        - ということはこの場合は、取り出したい値が常に最優先になるように実装するのが良さそう
  - 入力(nums)がk個未満の場合は何を返すのがいいのだろう
    - 一番低い点数が返ってくるのが自然に感じる
## 解答
- PriorityQueueを利用した実装
```java
class KthLargest {
    // k番目に大きい要素までだけを保持
    private Queue<Integer> kthLargestScores = new PriorityQueue<>();
    private int k;

    public KthLargest(int k, int[] nums) {
        this.k = k;
        for (int num : nums) {
            add(num);
        }
    }

    public int add(int val) {
        kthLargestScores.offer(val);
        if (kthLargestScores.size() > k) {
            kthLargestScores.poll();
        }
        return kthLargestScores.peek();
    }
}
```
- ソートされたArrayListを用いる方法
- ArrayListで全ての要素を保持して、k番目の要素を返す
  - 毎回ソートすると、コストが大きいので常にソートされた状態にして二分探索で挿入位置を求めるようにしておく
  - 最初書いた時comparatorを指定せず、List昇順のままにしていて意図した動作せず
    - 後から気づいたが、わざわざ降順にしなくても配列の末尾からk番目の要素を取得すればよかった
      - add()の返り値を取得する際に以下のようにする
        - `return kthLargestScores.get(kthLargestScores.size() - k);`
```java
class KthLargest {
    private List<Integer> kthLargestScores = new ArrayList<>();
    private int k;
    private Comparator<Integer> desc = Comparator.reverseOrder();

    public KthLargest(int k, int[] nums) {
        this.k = k;
        kthLargestScores.addAll(
                Arrays.stream(nums).boxed().sorted(desc).toList()
        );
    }

    public int add(int val) {
        int insertIndex = Collections.binarySearch(kthLargestScores, val, desc);
        if (insertIndex < 0) {
            insertIndex = -(insertIndex + 1);
        }
        kthLargestScores.add(insertIndex, val);
        // indexは0basedのため-1する
        return kthLargestScores.get(k - 1);
    }
}
```

# step2 他の方の解答を見る
- クラスのプロパティとして`topKScores`という命名が自然だと感じた
- コンストラクタに渡されるkが1以上であることをチェックしている実装もある
  - 確かにチェックしてある方が親切かも
## 解答
- 変数名を少し修正、コンストラクで引数チェックを追加
```java
class KthLargest {
    private Queue<Integer> topKScores = new PriorityQueue<>();
    private int k;

    public KthLargest(int k, int[] nums) {
        if (k <= 0) {
            throw new IllegalArgumentException("k must be larger than 0. k : " + k);
        }
        this.k = k;
        for (int num : nums) {
            add(num);
        }
    }

    public int add(int val) {
        topKScores.offer(val);
        if (topKScores.size() > k) {
            topKScores.poll();
        }
        return topKScores.peek();
    }
}
```

# step3 3回ミスなく書く
## 解答
- PriorityQueueを使って実装
  - Step2と同様の感じ
    - 今回の入力の制限に沿って、入力値チェックは省略
```java
class KthLargest {
    private Queue<Integer> topKScores = new PriorityQueue<>();
    private int k;

    public KthLargest(int k, int[] nums) {
        this.k = k;
        for (int num : nums) {
            add(num);
        }
    }

    public int add(int val) {
        topKScores.offer(val);
        if (topKScores.size() > k) {
            topKScores.poll();
        }
        return topKScores.peek();
    }
}
```
