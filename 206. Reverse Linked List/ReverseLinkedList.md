# step1 何も見ずに解く
## 解答
- Stackを使って実装
  - Stackを使わなくても書けそうな気がするが、どうしてStackのセクションにあるのだろう。
- 当初、末尾の要素のnextをnullにするのを忘れていた。
  - 最後の要素だけが問題になるので、ループ完了後に`newTail.next = null;`としているが、ループの中で上書きした方が読み手にとって自然だろうか。
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
    public ListNode reverseList(ListNode head) {
        if (head == null) {
            return null;
        }
        // Stackとして使用
        Deque<ListNode> nodeStack = new ArrayDeque<>();
        ListNode node = head;

        // 全ノードをStackに積む
        while (node != null) {
            nodeStack.push(node);
            node = node.next;
        }
        
        // 逆順のLinked Listを生成
        ListNode newHead = nodeStack.pop();
        ListNode newTail = newHead;
        while (!nodeStack.isEmpty()) {
            newTail.next = nodeStack.pop();
            newTail = newTail.next;
        }
        // 元の順序の時の参照が残っているので初期化
        newTail.next = null;
        return newHead;
    }
}
```
- 同様の実装で入力非破壊版も書いてみる
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null) {
            return null;
        }
        // Stackとして使用
        Deque<int> valStack = new ArrayDeque<>();
        ListNode node = head;
        
        while (node != null) {
            valStack.push(node.val);
            node = node.next;
        }
        ListNode newHead = new ListNode(valStack.pop());
        ListNode newTail = newHead;
        while (!valStack.isEmpty()) {
            newTail.next = new ListNode(valStack.pop());
            newTail = newTail.next;
        }
        return newHead;
    }
}
```
- Stackを使わない実装
- prev、current、nextの対比とするのが分かりやすいかと思ったが、prevをreturnしているのがちょっと気持ち悪い
- 計算量
  - 時間計算量: O(n)
  - 空間計算量: O(1)
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null) {
            return null;
        }
        ListNode current = head;
        ListNode prev = null;
        while (current != null) {
            ListNode next = current.next;
            current.next = prev;
            prev = current;
            current = next;
        }
        return prev;
      }
}
```

# step2 他の方の解答を見る
- 7問目まで進めてやっと書き方の幅に思い至る
  - ここで手が止まってしまった
  - 網羅的にしようとするとキリがないな
    - とりあえず気に入ったものを記す
  - 色々な書き方の方針を自然言語で理解することを意識してみる
## 解答
- step1で書いた実装の変数名を修正
  - `prev`→`newHead`とした方が分かりやすいか
    - step3では`reversedHead`にした
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null) {
            return null;
        }
        ListNode node = head;
        ListNode newHead = null;
        while (node != null) {
            ListNode next = node.next;
            node.next = newHead;
            newHead = node;
            node = next;
        }
        return newHead;
      }
}
```
- https://github.com/akmhmgc/arai60/pull/7/files
- 1ノードずつ逆順のLinked Listの先頭に繋いでいく
- 上の実装とやっていることはほぼ同じだが、すっきりした実装に
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode newHead = null;
        ListNode node = head;

        while (node != null) {
            newHead = new ListNode(node.val, newHead);
            node = node.next;
        }
        return newHead;
    }
}
```
- 再帰の実装
- https://github.com/colorbox/leetcode/pull/22/files
- 末尾再帰の実装
- 先頭から順に逆に繋ぎ直す。繋ぎ直した結果は第二引数で受け渡し
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        return reverseListHelper(head, null);
    }

  /**
   * 未処理のLinked Listから一つ取り出して、逆順のLinked Listの先頭に繋ぐ
   * @param head 未処理のLinked Listの先頭
   * @param newHead 逆順に繋ぎ直したLinked Listの先頭
   * @return 逆順に繋ぎ直したLinked Listの先頭
   */
  private ListNode reverseListHelper(ListNode head, ListNode newHead) {
        if (head == null) {
            return newHead;
        }
        ListNode next = head.next;
        head.next = newHead;
        return reverseListHelper(next, head);
    }
}
```
- https://github.com/nanae772/leetcode-arai60/pull/8/files
- 末尾ではなく処理の途中で再帰した実装。こちらの方は再帰の処理結果を引き継がなくていいので引数が1つで済む
  - 末尾で再帰の方が処理の流れがイメージしやすく感じた
- 入力のLinked Listの末尾から先頭に向かって、Linked Listを繋ぎ直す
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        return reverseListHelper(head);
    }

  private ListNode reverseListHelper(ListNode node) {
        if (node == null || node.next == null) {
            return node;
        }
        ListNode headReversed = reverseListHelper(node.next);
        ListNode tailReversed = node.next;
        tailReversed.next = node;
        node.next = null;
        return headReversed;
    }
}
```
# step3 3回ミスなく書く

## 解答
- 1ノードずつ逆順のLinked Listの先頭に繋いでいく
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode node = head;
        ListNode reversedHead = null;
        while(node != null) {
            ListNode next = node.next;
            node.next = reversedHead;
            reversedHead = node;
            node = next;
        }
        return reversedHead;
    }
}
```
