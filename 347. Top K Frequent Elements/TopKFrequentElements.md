# step1 何も見ずに解く
- numsからMap<値, 頻度>を作成して、そこから頻度が多いk件を取り出せば良い
    - やりたいことは簡単ではあるのだが、PriorityQueueの優先度をMapのvalue降順にする書き方が分からず調べた
        - 自分でComparator書かなくても標準であるはずと思い確認すると`Map.Entry.comparingByValue`がある
        - https://docs.oracle.com/javase/jp/8/docs/api/java/util/Map.Entry.html#comparingByValue--
    - PriorityQueueを使わなくても`numToCount.entrySet()`を
## 解答

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> numToCount = new HashMap<>();
        for (int num : nums) {
            numToCount.merge(num, 1, Integer::sum);
        }

        PriorityQueue<Map.Entry<Integer, Integer>> frequentNums
                = new PriorityQueue<>(Map.Entry.comparingByValue(Comparator.reverseOrder()));
        frequentNums.addAll(numToCount.entrySet());

        int[] result = new int[k];
        for (int i = 0; i < k; i++) {
            result[i] = frequentNums.poll().getKey();
        }
        return result;
    }
}
```
- PriorityQueueに不要な値を溜めない版
```java
import java.util.Map;
import java.util.PriorityQueue;
import java.util.Comparator;

class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> numToCount = new HashMap<>();
        for (int num : nums) {
            numToCount.merge(num, 1, Integer::sum);
        }

        PriorityQueue<Map.Entry<Integer, Integer>> frequentNums
                = new PriorityQueue<>(Map.Entry.comparingByValue());
        for (Map.Entry<Integer, Integer> entry : numToCount.entrySet()) {
            frequentNums.offer(entry);
            if (frequentNums.size() > k) {
                frequentNums.poll();
            }
        }

        int[] result = frequentNums.stream()
                .mapToInt(Map.Entry::getKey).toArray();
        return result;
    }
}
```
- PriorityQueueを使わず、ソートでも書いてみる
    - `numToCountEntries.get(i).getKey();`で`IndexOutOfBoundsException`するのを避けたいと思って少し考えたら最初にreturnしておけばよかった
    - `numToCountEntries`の辺りの命名はどうすればいいか判断がつかない
        - ソートされた値を保持する変数なのでそれを名前に含めたいが、宣言時点で未ソートだしな
        - stream APIを使って変数を経由せずに書くこともできるが、やや読み辛い仕上がりになるだろう
```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        if (nums.length <= k) {
            return nums;
        }
        Map<Integer, Integer> numToCount = new HashMap<>();
        for (int num : nums) {
            numToCount.merge(num, 1, Integer::sum);
        }

        List<Map.Entry<Integer, Integer>> numToCountEntries = new ArrayList<>(numToCount.entrySet());
        numToCountEntries.sort(Map.Entry.comparingByValue(Comparator.reverseOrder()));

        int[] result = new int[k];
        for (int i = 0; i < k; i++) {
            result[i] = numToCountEntries.get(i).getKey();
        }
        return result;
    }
}
```

# step2 他の方の解答を見る
- 同率の値があった場合について考えられていなかった
    - PriorityQueueを使った上記の実装だと同率のどの値が出力されるか定まらないっぽい
        - https://docs.oracle.com/javase/jp/8/docs/api/java/util/PriorityQueue.html
        - >  複数の要素が最小の値に結び付けられている場合、先頭はこれらの要素の1つになります。
- QuickSelect
    - クイックソートは聞いたことがあったが、クイックセレクトは初めて知った
    - クイックソートの要領で、値を左右によせるがソートまではしない
- BucketSort
  - 同じ値をBucketに詰めて、それらを連結してソート
    - 時間計算量を空間計算量に転化していると考えたらいいのだろうか
## 解答
- https://leetcode.com/problems/kth-largest-element-in-a-stream/editorial/
- https://github.com/seal-azarashi/leetcode/pull/9
    - Editorialなどを参考にしてとりあえずQuickSelectの実装
      - 腹落ちしきっていないので、時間を置いてからまた戻ってきたい
    - Java16から`Record`を使ってイミュータブルな値の集合を定義できる
      - https://docs.oracle.com/javase/jp/16/docs/api/java.base/java/lang/Record.html
      - https://docs.oracle.com/javase/specs/jls/se16/html/jls-8.html#jls-8.10
```java
class Solution {

    private record NumAndFrequency(int num, int frequency) {}

    public int[] topKFrequent(int[] nums, int k) {
        // 値ごとの出現頻度を数える
        Map<Integer, Integer> numToFrequency = new HashMap<>();
        for (int num : nums) {
            numToFrequency.merge(num, 1, Integer::sum);
        }

        // 値と出現頻度の組み合わせを配列にする
        int n = numToFrequency.size();
        NumAndFrequency[] numAndFrequencies = numToFrequency.entrySet()
                .stream()
                .map(v -> new NumAndFrequency(v.getKey(), v.getValue()))
                .toArray(NumAndFrequency[]::new);

        // 頻度の高い要素を配列の(n - k)番目からn番目に集める
        quickselect(numAndFrequencies, 0, n - 1, n - k);

        // 頻度の高い(n - k)番目からn番目の要素のnumを取り出し配列にして返す
        return Arrays.stream(Arrays.copyOfRange(numAndFrequencies, n - k, n))
                .map(NumAndFrequency::num)
                .mapToInt(Integer::intValue).toArray();
    }

    private void quickselect(NumAndFrequency[] numAndFrequencies, int left, int right, int kSmallest) {
        // 要素が一つしかない場合
        if (left == right) return;

        // (n - k)番目に頻度が高い値(kSmallestと一致する値)を選べればベストだが
        // 分からないので、ピボットをランダムに選ぶ
        Random random = new Random();
        int pivotIndex = left + random.nextInt(right - left + 1);

        // ピボットに指定された要素を基準に、頻度が低いものを左に、高いものを右に寄せる
        // 返されるpivotIndexは元々pivotIndexにあった要素の新しい位置
        pivotIndex = partition(numAndFrequencies, left, right, pivotIndex);

        // topKの要素が右に寄せられたら、終了
        if (kSmallest == pivotIndex) {
            return;
        }
        if (kSmallest < pivotIndex) {
            quickselect(numAndFrequencies, left, pivotIndex - 1, kSmallest);
        } else {
            quickselect(numAndFrequencies, pivotIndex + 1, right, kSmallest);
        }
    }

    private int partition(NumAndFrequency[] numAndFrequencies, int left, int right, int pivotIndex) {
        int pivotFrequency = numAndFrequencies[pivotIndex].frequency;
        // ピボットの要素を一旦右端においておく
        swap(numAndFrequencies, pivotIndex, right);
        int storeIndex = left;

        // ピボットより、頻度が低い要素を左端によせていく
        for (int i = left; i <= right; i++) {
            if (numAndFrequencies[i].frequency < pivotFrequency) {
                swap(numAndFrequencies, storeIndex, i);
                storeIndex++;
            }
        }

        // 右端に退避していたピボットの要素を頻度が低い要素群の右隣に配置
        // 頻度が低い要素群→ピボット要素→頻度が高い要素群
        swap(numAndFrequencies, storeIndex, right);

        // 並べ替え後のピボット要素の位置を返す
        return storeIndex;
    }

    private void swap(NumAndFrequency[] numAndFrequencies, int a, int b) {
        NumAndFrequency temp = numAndFrequencies[a];
        numAndFrequencies[a] = numAndFrequencies[b];
        numAndFrequencies[b] = temp;
    }
}
```

# step3 3回ミスなく書く

## 解答
- Step1のPriorityQueueを使った実装とほぼ一緒
  - `numToCountEntry`としているが、命名がいまいちな感じ
```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        if (nums.length <= k) {
            return nums;
        }

        Map<Integer, Integer> numToCount = new HashMap<>();
        for (int num : nums) {
            numToCount.merge(num, 1, Integer::sum);
        }

        PriorityQueue<NumAndFrequency> topKNumToCountEntry
                = new PriorityQueue<>(Map.Entry.comparingByValue());

        for (Map.Entry<Integer, Integer> numToCountEntry : numToCount.entrySet()) {
            topKNumToCountEntry.offer(numToCountEntry);
            if (topKNumToCountEntry.size() > k) {
                topKNumToCountEntry.poll();
            }
        }
        int[] result = new int[k];
        for (int i = 0; i < k; i++) {
            result[i] = topKNumToCountEntry.poll().getKey();
        }
        return result;
    }
}
```
