# step1 何も見ずに解く

## 解答
- Javaの標準ライブラリの機能で変換や積集合を取得することができるはずと思ったらやっぱりあった
    - stream APIを多用しているがこれくらいだったら違和感なく読めるか?
    - `addAll()`する中身をHashSetのコンストラクタに直接指定できるが、ちょっと長くて読みにくい気もする
```java
import java.util.*;

class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        Set<Integer> intersection = new HashSet<>();
        intersection.addAll(Arrays.stream(nums1).boxed().toList());
        intersection.retainAll(Arrays.stream(nums2).boxed().toList());
        return intersection.stream().mapToInt(Integer::intValue).toArray();
    }
}
```
- Setを使わず、`List`と`distinct`でも書ける
```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        List<Integer> intersection = new ArrayList<>(Arrays.stream(nums1).distinct().boxed().toList());
        intersection.retainAll(Arrays.stream(nums2).boxed().toList());
        return intersection.stream().mapToInt(Integer::intValue).toArray();
    }
}
```
- 素朴に書いてみる
```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        // nums1のsetを作る
        Set<Integer> nums1Set = new HashSet<>();
        for (int num1 : nums1) {
            nums1Set.add(num1);
        }

        List<Integer> intersection = new ArrayList<>();
        for (int num2 : nums2) {
            if (nums1Set.contains(num2)) {
                intersection.add(num2);
                nums1Set.remove(num2);
            }
        }

        // Listの中身を配列に変換
        int[] result = new int[intersection.size()];
        for (int i = 0; i < intersection.size(); i++) {
            result[i] = intersection.get(i);
        }
        return result;
    }
}
```
# step2 他の方の解答を見る
- 入力として渡される配列の長さをチェックして、短い方をSetにするという視点は抜けていた
    - `if (nums1.length < nums2.length) intersection(nums2, nums1);`みたいな書き方はシンプルでいいなと思った
- 入力がソートされていた場合などのケースについては考えれていなかった
## 解答
- https://github.com/nanae772/leetcode-arai60/pull/14/files
- 入力が両方ともソートされていた場合
- 先頭から見ていって、一致していれば結果に含める
  - 最初に重複排除せずintersectionをSetで書くのもあり
  - あるいはwhileループ中の条件を少し工夫するのもあり
```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        int[] uniqueNums1 = Arrays.stream(nums1).distinct().toArray();
        int[] uniqueNums2 = Arrays.stream(nums2).distinct().toArray();
        Arrays.sort(uniqueNums1);
        Arrays.sort(uniqueNums2);

        List<Integer> intersection = new ArrayList<>();
        int i = 0;
        int j = 0;
        while (i < uniqueNums1.length || j < uniqueNums2.length) {
            if (uniqueNums1[i] == uniqueNums2[j]) {
                intersection.add(uniqueNums1[i]);
                i ++;
                j ++;
                continue;
            }
            if (uniqueNums1[i] < uniqueNums2[j]) {
                i ++;
            } else {
                j ++;
            }
        }
        return intersection.stream().mapToInt(Integer::intValue).toArray();
    }
}
```
- https://github.com/akmhmgc/arai60/pull/10/files
- 片方の配列がソートされていれば二分探索でも書ける
```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        Arrays.sort(nums1);
        Set<Integer> intersection = new HashSet<>();
        for (int num : nums2) {
            int intersectionIndex = Arrays.binarySearch(nums1, num);
            if (intersectionIndex >= 0) {
                intersection.add(nums1[intersectionIndex]);
            }
        }
        return intersection.stream().mapToInt(Integer::intValue).toArray();
    }
}
```
# step3 3回ミスなく書く
- IDEによる補完が無いと書けないStream APIの練習も兼ねてStream APIで書く
## 解答
```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        Set<Integer> intersection = new HashSet<>();
        intersection.addAll(Arrays.stream(nums1).boxed().toList());
        intersection.retainALl(Arrays.stream(nums2).boxed().toList());
        return intersection.stream().mapToInt(Integer::intValue).toArray();
    }
}
```
