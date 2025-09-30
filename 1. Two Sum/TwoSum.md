# step1 何も見ずに解く

## 解答
- hashMapを使った実装
- partnerという単語がパッと思い浮かんだが分かりやすいだろうか
- `numToIndex.get()`した値をnullチェックするより、`numToIndex.containsKey()`する方が読み手にとって親切だろうか
  - hashの計算を2回するのがもったいないように感じて最初から`get()`する形で書いた
- 計算量
  - 時間計算量: O(n)
  - 空間計算量: O(n)
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> numToIndex = new HashMap<>();
        for(int i = 0; i < nums.length; i++) {
            int num = nums[i];
            int partnerNum = target - num;
            Integer partnerIndex = numToIndex.get(partnerNum);
            if (partnerIndex != null) {
                return new int[]{partnerIndex, i};
            }
            numToIndex.put(num, i);
        }
        return new int[]{};
    }
}
```
- 二重にforループを使った総当たり実装
  - 最初に書いた時、ループのindexが重なっている時に`continue`するのを忘れていた
- 計算量
    - 時間計算量: O(n^2)
    - 空間計算量: O(1)
    - よく考えたら空間計算量的には二重ループの方が優れているか
    - 要素数が一定より少なければhashMapの生成などが不要なためこちらの方が高速になるだろう
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        for(int i = 0; i < nums.length; i++) {
            for(int j = 0; j < nums.length; j++) {
                if (i == j) {
                    continue;
                }
                if (target == nums[i] + nums[j]) {
                    return new int[]{i, j};
                }
            }
        }
        return new int[]{};
    }
}
```
# step2 他の方の解答を見る
## 解答
- https://github.com/akmhmgc/arai60/pull/8/files
- よく考えたら内側のループの始点は`i + 1`でよかった
  - それ以前の値は確認済みのため確認不要
- また該当がなかった場合は仮の値を何かしら入れておくのが良いだろう
  - 到達することは無いはずだから適当でいいかくらいにstep1では考えてた
  - nullか-1を入れておくのが一般的だろうか
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
          for (int j = i + 1; j < nums.length; j++) {
              if (target == nums[i] + nums[j]) {
                  return new int[]{i, j};
              }
          }
        }
        // ここに到達することは無いはず
        return new int[]{-1, -1};
    }
}
```
- hashMapを使った実装において処理の流れは一緒だが以下のような書き方も想定できたらよかった
  - ターゲットとある値の差分を`complement`とする
  - Mapのキーとして「走査済みの値」ではなく「ターゲットと走査した値の差分」をセットする
    - 後で引き算するか先に引き算するかの違い。個人的には今回書いている前者の方が分かりやすいように思う
- その他二分探索を使用した書き方も考えられる
  - しかし二分探索するためにはソート&元のインデックスを保持する必要があるためこの場合は不適なように感じる
  - この問題の条件で二分探索の方が適する入力のケースってあるだろうか?
# step3 3回ミスなく書く
- 所要時間: 4分程度
## 解答
- HashMapを使った実装
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> numToIndex = new HashMap<>();
        for(int i = 0; i  < nums.length; i++) {
            int num = nums[i];
            int partnerNum = target - num;
            Integer partnerIndex = numToIndex.get(partnerNum);
            if (partnerIndex != null) {
                return new int[]{partnerIndex, i};
            }
            numToIndex.put(num, i);
        }
        return new int[]{-1, -1};
    }
}
```
