# step1 何も見ずに解く
- Javaのint、Integerの比較についておさらい
  - int同士の比較 → 常に値で比較
    - `==`で比較すればよい
  - Integer同士の比較 → 常にオブジェクトで比較
    - 値を比較したいなら`Objects.equals()`などで比較
    - ただし -128 ～ 127 の範囲の整数はキャッシュされてるため、同じオブジェクトを参照しており `==` が true になる。
  - intとIntegerの比較 → 常にintで比較
    - Integerがunboxingされて、int同士の比較として実行される
## 解答
- もっとシンプルな解法があるはずと思い、挑戦するも挫折
    - とりあえず動くものを一旦実装
        - でもこれって自分の手慣れた道具(MapとかSetとか)を使うことありきの実装になっていて、不健全
- 計算量
    - 時間計算量: O(n)
    - 空間計算量: O(n)
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

        // 各ノードの数を数える
        Map<Integer, Integer> uniqueValCounter = new HashMap<>();
        ListNode node = head;
        while (node != null) {
            // Mapに値がセットされていなければ1をセット
            // Mapに値がセットされていればインクリメントしてセット
            // 以下のコードと等価
            // uniqueValCounter.compute(node.val, (k, v) -> v == null ? 1 : v + 1);
            uniqueValCounter.merge(node.val, 1, Integer::sum);
            node = node.next;
        }

        // 重複していない値をListに詰める
        List<Integer> uniqueVals = new ArrayList<>();
        uniqueValCounter.forEach((k, v) -> {
            // 重複していない値だけ、Listに詰める
            if (v == 1) {
                uniqueVals.add(k);
            }
        });
        Collections.sort(uniqueVals);

        // 新しいLinkedListを生成
        ListNode newHead = null;
        ListNode newTail = null;
        for (int val : uniqueVals) {
            if (newHead == null) {
                newHead = new ListNode(val);
                newTail = newHead;
            } else {
                newTail.next = new ListNode(val);
                newTail = newTail.next;
            }
        }
        return newHead;
    }
}
```
- stackを使った実装
    - これまでStackってほとんど使ったことがなかったが、書いてみるとすんなり書けた
    - Mapを使った実装よりも随分と自然な実装になった
- 計算量
    - 時間計算量: O(n)
    - 空間計算量: O(n)
```java
import java.util.Stack;

class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        Stack<Integer> stack = new Stack<>();
        ListNode node = head;

        while (node != null) {
            if (stack.empty() || stack.peek() != node.val) {
                stack.push(node.val);
                node = node.next;
            } else {
                int val = stack.pop();
                while (node != null && val == node.val) {
                    node = node.next;
                }
            }
        }

        // ListNode head = null;
        ListNode tail = null;
        while (!stack.empty()) {
            tail = new ListNode(stack.pop(), tail);
        }
        return tail;
    }
}
```

# step2 他の方の解答を見る
- 参考文献
    - https://github.com/SanakoMeine/leetcode/pull/5/files
    - https://github.com/Kyosuke-Asaki/leetcode/pull/5/files
    - https://github.com/fuga-98/arai60/pull/5/files
- 番兵: `sentinel`というタームをはじめて知る
    - 最初混乱してしまったのは、ループの中で全て処理しようとし過ぎていたことも一因に思う
        - 番兵を用いることで条件分岐を減らすことでシンプルに実装できる
- 計算量
    - 時間計算量: O(n)
    - 空間計算量: O(1)
## 解答
- 最初に見たこの解法、理解するまでにとても時間がかかってしまった
    - この実装だと`tail.next`に重複ノードがセットされるのではと懸念していたが次のループで`tail`ごと更新することで問題が起きない
        - そこを理解するまでに時間がかかった
```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        // ダミー、セットされている値に意味は無い
        ListNode dummy = new ListNode(Integer.MIN_VALUE, head);
        ListNode tail = dummy;
        ListNode node = head;

        while (node != null && node.next != null) {
            int val = node.val;
            // 次のノードと異なる場合
            if (val != node.next.val) {
                // nextに誤ったノードがセットされていても、ここでtail丸ごと上書き
                tail = node;
                node = node.next;
                continue;
            }
            // 同じ値のノードが連続する場合、スキップしていく
            while (node != null && val == node.val) {
                node = node.next;
            }
            // 重複があるノードの可能性があるが一旦セット
            // 重複していた場合、次のループでtailごと上書きされる
            tail.next = node;
        }
        return dummy.next;
    }
}
```
- 外側のループ開始時、nodeは連続する同じvalを持つnodeの先頭であることが常に保たれる
    - これが一番分かりやすく感じた
```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode dummy = new ListNode(Integer.MIN_VALUE, head);
        ListNode tail = dummy;
        ListNode node = head;

        // 外側のループ開始時、nodeは連続する同じvalを持つnodeの先頭であることが常に保たれる
        while (node != null) {
            if (node.next == null || node.val != node.next.val) {
                tail.next = new ListNode(node.val);
                tail = tail.next;
            } else {
                // 重複したノードを省き、次のvalを持つノードが現れるまでスキップ
                while (node.next != null && node.val  == node.next.val) {
                    node.next = node.next.next;
                }
            }
            node = node.next;
        }
        return dummy.next;
    }
}
```
- 再起
    - シンプルだけど、パッと見では挙動をイメージできなかった
        - 末尾の要素から繋げていくということか
```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        if (head.val != head.next.val) {
            head.next = deleteDuplicates(head.next);
            return head;
        } else {
            while (head.next != null && head.val == head.next.val) {
                head.next = head.next.next;
            }
            return deleteDuplicates(head.next);
        }
    }
}
```
# step3 3回ミスなく書く
- 一番シンプルで分かりやすいと感じた実装を採用
- 所要時間 平均3分程度
## 解答
```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode dummy = new ListNode(Integer.MIN_VALUE, null);
        ListNode tail = dummy;
        ListNode node = head;

        while(node != null) {
            if (node.next == null || node.val != node.next.val) {
                tail.next = new ListNode(node.val);
                tail = tail.next;
            } else {
                while(node.next != null && node.val == node.next.val) {
                    node.next = node.next.next;
                }
            }
            node = node.next;
        }
        return dummy.next;
    }
}
```
