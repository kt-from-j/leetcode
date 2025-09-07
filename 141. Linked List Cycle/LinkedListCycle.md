# step1 何も見ずに解く
問題文の説明を理解するのに時間を要す。
何のために`pos`という概念があるのか。
とりあえず循環を検知するコードを書いて動かすことで、実装のためというよりテストケースの説明のための概念であることを理解する。
この`pos`はpositionの略でしょうか?

## 解答
- 普通に書いたら書けたが、問題文を理解するのに時間がかかった。
- 普段IDEに頼りきっているので、leetcode上のただのエディタで書くとメソッド名などまごついた。
  - contains()だっけ、has()だっけみたいな。
- O(1)の解答方法は全く思いつかず。
- 計算量
  - 時間計算量: O(n)
  - 空間計算量: O(n)

```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public boolean hasCycle(ListNode head) {
        if(head == null) return false;
        // 巡回済みのnodeを保持するSet
        HashSet<ListNode> nodeSet = new HashSet<>();
        ListNode checkNode = head;
        while (checkNode.next != null) {
            if (nodeSet.contains(checkNode)) return true;
            nodeSet.add(checkNode);
            checkNode = checkNode.next;
        }
        return false;
    }
}
```

# step2 他の方の解答を見る
フロイドの循環検出法を知る。 循環小数の検出などにも使用するよう。
速度違いで2つのポインタを動かすことで、ループがあればいつか必ず追いつくということか。

## 解答
- leetcodeの画面で動かすと、フロイド法は0ms、Set使用時は4msとなる。
  - 時間計算量は同じO(n)なのにここまで差がつくのが直感に反した。
    - Setを使った場合、Hashの計算やメモリの割り当てなどで余計に時間がかかるからだろうか?
- 計算量
  - 時間計算量: O(n)
  - 空間計算量: O(1)

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        if(head == null) return false;
        ListNode fast = head;
        ListNode slow = head;
        while (fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
            if (fast == slow) return true;
        }
        return false;
    }
}
```

## 解答2
- Setを使った解法についてもstep1の変数名を少し改良して再実装

```java
public class Solution {
  public boolean hasCycle(ListNode head) {
    if (head == null) return false;
    HashSet<ListNode> visited = new HashSet<>();
    ListNode node = head;
    while (node.next != null) {
        if (visited.contains(node)) return true;
        visited.add(node);
        node = node.next;
    }
    return false;
  }
}
```

## step3 3回ミスなく書く
何回か書いたのでスラスラかける。
所要時間 2分程度
## 解答
```java
public class Solution {
    public boolean hasCycle(ListNode head) {
    if (head == null) return false;
    HashSet<ListNode> visited = new HashSet<>();
    ListNode node = head;
    while (node != null && node.next != null) {
        if (visited.contains(node)) return true;
        visited.add(node);
        node = node.next;
    }
    return false;
  }
}
```
