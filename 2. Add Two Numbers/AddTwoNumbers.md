# step1 何も見ずに解く

## 解答
- 愚直に書いてみる
  - 複数回に分けて`while`ループを回しているのを単純化したいが、パッと思いつかず
- 計算量
    - 時間計算量: O(n)
      - 入力のサイズは最大で100
      - JITコンパイルされていなくても1ループ辺り多く見積もっても10μ秒程度あれば処理できそう
      - 実行時間は2ms以下には収まりそう
    - 空間計算量: O(1)
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
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;
        boolean isCarryUp = false;
        // どちらのListNodeが長いか分からないので、
        // とりあえずl1でループを回す
        // このループ終了時点でl1は末尾まで到達
        while (l1 != null) {
            int l2Val = l2 != null ? l2.val : 0;
            int sum = l1.val + l2Val + carryUp(isCarryUp);
            isCarryUp = false;
            if (sum > 9) {
                isCarryUp = true;
                sum = sum - 10;
            }
            tail.next = new ListNode(sum);
            tail = tail.next;
            l1 = l1.next;
            if (l2 != null) {
                l2 = l2.next;
            }
        }

        // l2の方が長かった場合は、
        // 繰り上がりに留意してl2のノードを連結
        while (l2 != null) {
            int sum = l2.val + carryUp(isCarryUp);
            isCarryUp = false;
            if (sum > 9) {
                isCarryUp = true;
                sum = sum - 10;
            }
            tail.next = new ListNode(sum);
            tail = tail.next;
            l2 = l2.next;
        }

        // 最後のノードが繰り上がりの場合、末尾にノードを追加
        if (isCarryUp) {
            tail.next = new ListNode(1);
        }
        return dummy.next;
    }

    // 下位の桁で繰り上がりがあった場合は、1を返す
    private int carryUp(Boolean isCarryUp) {
        return isCarryUp ? 1 : 0;
    }
}
```

# step2 他の方の解答を見る
- 参考
  - https://github.com/fuga-98/arai60/pull/6/
  - https://github.com/fhiyo/leetcode/pull/5
## 解答
- 単一の`while`で処理する実装
  - intの変数`carry`に直接繰り上がりを足せば、フラグを立てる必要もないのか
```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode();
        ListNode tail = dummy;
        int carry = 0;
        while (carry != 0 || l1 != null || l2 != null) {
            if (l1 != null) {
                carry += l1.val;
                l1 = l1.next;
            }
            if (l2 != null) {
                carry += l2.val;
                l2 = l2.next;
            }
            tail.next = new ListNode(carry % 10);
            tail = tail.next;
            carry = carry / 10;
        }
        return dummy.next;
    }
}
```
- 再帰を使った実装
- ついでに、nullチェックを関数化した実装も試してみる。
```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        return addTwoNumbers(l1, l2, 0);
    }

    private ListNode addTwoNumbers(ListNode l1, ListNode l2, int carry) {
        if (carry == 0 && l1 == null && l2 == null) {
            return null;
        }
        
        carry = carry + getValOrDefault(l1) + getValOrDefault(l2);
        l1 = getNextOrNull(l1);
        l2 = getNextOrNull(l2);
        ListNode nextNode = addTwoNumbers(l1, l2, carry / 10);
        return new ListNode(carry % 10, nextNode);
    }
    
    private int getValOrDefault(ListNode node) {
        return node != null ? node.val : 0;
    }

    private ListNode getNextOrNull(ListNode node) {
        return node == null ? null : node.next;
    }
}
```

# step3 3回ミスなく書く
- 単一の`while`を使った実装で書く
## 解答
```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode();
        ListNode node = dummy;
        int carry = 0;

        while (carry != 0 || l1 != null || l2 != null) {
            if (l1 != null) {
                carry += l1.val;
                l1 = l1.next;
            }
            if (l2 != null) {
                carry += l2.val;
                l2 = l2.next;
            }
            node.next = new ListNode(carry % 10);
            node = node.next;
            carry = carry / 10;
        }
        return dummy.next;
    }
}
```
