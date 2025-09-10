# step1 何も見ずに解く

## 解答
- Set使わなくても書けるはずだが、パッと思いついたのでSetを使って実装
  - 意図は理解しやすい気はするが、流石に無駄が多いか
- 計算量
    - 時間計算量: O(n)
    - 空間計算量: O(n)
    - これって2回ループ回しているからO(2n)とすべきですか?
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) {
            return null;
        }

        // 重複を省いたvalの配列を作成
        ListNode node = head;
        HashSet<Integer> uniqueValManager = new HashSet<>();
        ArrayList<Integer> uniqueVals = new ArrayList<>();
        while (node != null) {
            if (!uniqueValManager.contains(node.val)) {
                uniqueValManager.add(node.val);
                uniqueVals.add(node.val);
            }
            node = node.next;
        }

        // valの配列を元に繋ぎ直す
        ListNode prevNode = head;
        prevNode.next = null;
        // 配列の先頭の値は追加不要なので 1 からスタート
        for (int i = 1; i < uniqueVals.size(); i++) {
            prevNode.next = new ListNode(uniqueVals.get(i));
            prevNode = prevNode.next;
        }
        return head;
    }
}
```

# step2 他の方の解答を見る
当初書こうとした時はprevNode保持してとか考えたが、`node.next.next`使えばよかった
## 解答
- 1重ループ版
- 計算量
    - 時間計算量: O(n)
    - 空間計算量: O(1)
- `continue`で書くか、`if-else`で書くか悩む
  - `continue;`を使い、`while`ループの末尾で必ず`node`を更新する方が事故が起こりづらそう
```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) {
            return null;
        }

        ListNode node = head;
        while (node != null && node.next != null) {
            if (node.val == node.next.val) {
                node.next = node.next.next;
                continue;
            }
            node = node.next;
        }
        return head;
    }
}
```

## 解答2
- 2重ループ版
```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) {
            return null;
        }

        ListNode node = head;
        while (node != null) {
            // valが重複するならnextを上書き
            while (node.next != null && node.val == node.next.val) {
                node.next = node.next.next;
            }
            node = node.next;
        }
        return head;
    }
}
```

# step3 3回ミスなく書く
- 1重ループの方が好みなのでそちらで実装。
## 解答
```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) {
            return null;
        }

        ListNode node = head;
        while (node != null && node.next != null) {
            if (node.val == node.next.val) {
                node.next = node.next.next;
                continue;
            }
            node = node.next;
        }
        return head;
    }
}
```
