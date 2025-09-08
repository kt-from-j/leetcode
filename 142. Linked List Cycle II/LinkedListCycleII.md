# step1 何も見ずに解く

## 解答
- 147. Linked List Cycleそのままなので、何も悩まずかけた。
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
    public ListNode detectCycle(ListNode head) {
        if (head == null) {
            return null;
        }

        HashSet<ListNode> visitedNodes = new HashSet<>();
        ListNode node = head;

        while (node != null) {
            if (visitedNodes.contains(node)) {
                return node;
            }
            visitedNodes.add(node);
            node = node.next;
        }
        return null;
    }
}
```

## 解答2
- [Rでフロイドの循環検出法を可視化する](https://qiita.com/nozma/items/bfa3e089cd432b74c10d) で始点検出方法を理解し、実装してみる
- 運用するプログラムの場合、フロイドの循環検出法の概要からコメントを書くが、この場合どこまで書くか
- 計算量
  - 時間計算量: O(n)
  - 空間計算量: O(1)
```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        if (head == null) {
            return null;
        }

        // ループを検知
        ListNode slowNode = head;
        ListNode fastNode = head;
        while (fastNode != null && fastNode.next != null) {
            slowNode = slowNode.next;
            fastNode = fastNode.next.next;
            // 衝突を検知したら中断
            if (slowNode == fastNode) {
                break;
            }
        }

        if (fastNode == null || fastNode.next == null) {
            return null;
        }

        // ループの始点を探索
        // fastNodeを先頭に戻し、衝突するまで進める。
        // 衝突した地点がループの始点
        fastNode = head;
        while (fastNode != slowNode) {
            slowNode = slowNode.next;
            fastNode = fastNode.next;
        }
        return fastNode;
    }
}
```


# step2 他の方の解答を見る
Setの解法についてはあまり変更する余地が思いつかなかったので、フロイドの循環検出法で実装
## 解答
- ループ検知部分を関数化してみる。
  - collisionがパッと思い浮かんだのですがcollisionは違和感あるでしょうか?
- step1だと最初に定義した変数を使い回していたが、ループ始点検知のための処理の部分で再定義した方が意味がイメージしやすいなと思い真似してみる
```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        if (head == null) {
            return null;
        }

        ListNode collisionNode = findCollisionNode(head);
        if (collisionNode == null) {
            return null;
        }
        
        ListNode fromStart = head;
        ListNode fromCollision = collisionNode;
        
        while (fromStart == fromCollision) {
            fromStart = fromStart.next;
            fromCollision = fromCollision.next;
        }
        return fromStart;
    }
    
    // 衝突したノードを返す、衝突しなければnullを返す
    private ListNode findCollisionNode(ListNode node) {
        ListNode fastNode = node;
        ListNode slowNode = node;

        while (fastNode != null && fastNode.next != null) {
            fastNode = fastNode.next.next;
            slowNode = slowNode.next;

            if (fastNode == slowNode) {
                return fastNode;
            }
        }
        return null;
    }
}
```

# step3 3回ミスなく書く
Set解法で。
141と合わせて何度も書いたのでスラスラ書ける。
step1と同様のコード。
所要時間 2分足らず
## 解答
```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        if (head == null) {
            return null;
        }

        HashSet<ListNode> visitedNodes = new HashSet<>();
        ListNode node = head;

        while (node != null) {
            if (visitedNodes.contains(node)) {
                return node;
            }
            visitedNodes.add(node);
            node = node.next;
        }
        return null;
    }
}
```
