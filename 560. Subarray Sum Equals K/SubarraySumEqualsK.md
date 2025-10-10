# step1 何も見ずに解く
- 最初subArrayの定義を勘違いしていて少し戸惑った
  - `Input: nums = [1,1,1]` が `k = 2` になることに違和感を覚えて確認
    - インデックスで言うと`[0, 1]` `[0, 2]` `[1, 2]`でk=3になるのではと思った
  - SubArrayの定義は「元の配列から連続した要素の範囲を切り出したもの」
  - 当初イメージしていたものはSubSequenceと言われるらしい
    - SubSequenceの例: `[1, 2, 3, 4, 5]` → `[1, 3, 5]`
## 解答
- やりたいことから考えると累積和を保持してうまく活用すればいいのだと思うが、どうすればいいのか分からなかった
- 結局、愚直に全て足すことしか思い浮かばず
  - 一応これでもAcceptされたが1秒以上かかるようだ
  - numsに0や負の値が含まれていなければ内側のループをもっと早く打ち切れるのだが
```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int resultCount = 0;
        for (int i = 0; i < nums.length; i++) {
            int sum = nums[i];
            int j = i;
            while (true) {
                if (sum == k) {
                    resultCount++;
                }
                j++;
                if (j == nums.length) {
                    break;
                }
                sum += nums[j];
            }
        }
        return resultCount;
    }
}
```

# step2 他の方の解答を見る
- https://github.com/Shoichifunyu/shofun/pull/11/files#r2365544792
  - これをみて累積和をどう活用したらいいかを理解した
    - インデックスA、B(A < B)を仮定して`Bまでの累積和` - `Aまでの累積和`と`k`を比較すれば累積和==kの区間が存在することを発見できる
      - ひっくり返して差を求めるという発想がパッとでてこなかった
      - あとMapに`{0: 1}`をセットする意味にしばらく悩む
        - これが無いと配列の先頭からの区間和がkになるケースを見逃してしまう
  - この問題の場合は、numsに0以下の値が存在し重複した値が生じ得るためMapで出現回数を持つ
## 解答
```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        // 累積和の出現回数を記録するマップ
        Map<Integer, Integer> prefixSumToCount = new HashMap<>();
        prefixSumToCount.put(0, 1);
        
        int prefixSum = 0;
        int resultCount = 0;
        
        for (int num : nums) {
            prefixSum += num;
            int target = prefixSum - k;
            if (prefixSumToCount.containsKey(target)) {
                resultCount += prefixSumToCount.get(target);
            }
            prefixSumToCount.merge(prefixSum, 1, Integer::sum);
        }
        return resultCount;
    }
}
```

# step3 3回ミスなく書く
- 累積和を用いた書き方で実装、Step2と同様のコード
  - この問題を通じて累積和の活用方法が少し分かった
## 解答

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        Map<Integer, Integer> prefixSumToCount = new HashMap<>();
        prefixSumToCount.put(0, 1);
        Integer prefixSum = 0;
        Integer resultCount = 0;
        
        for (Integer num : nums) {
            prefixSum += num;
            Integer target = prefixSum - k;
            if (prefixSumToCount.containsKey(target)) {
                resultCount += prefixSumToCount.get(target);
            }
            prefixSumToCount.merge(prefixSum, 1, Integer::sum);
        }
        return resultCount;
    }
}
```
