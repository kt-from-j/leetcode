# step1 何も見ずに解く

## 解答
- Stackを使って実装。
  - JavaではLIFOのデータを扱うのにStackよりArrayDequeを使用することが推奨される
- 昔同じようなコードを書いたことがあったので実装方針はすぐ思いついた
  - 開き括弧は積む、閉じ括弧はpeekと比較して対応する括弧なら消し込む
  - 最後に空だったらtrue
- Type Scriptのように`for-of`で文字列→文字を取り出せるかと思ったが、Javaだとできなかった
  - というかJavaで文字を取り出す場合、StringではなくCharacterになるのか
- 本当は括弧の対応をEnumで表すのがいいのかもしれない
- 計算量
  - 時間計算量: O(n)
  - 空間計算量: O(n)
```java
class Solution {
    public boolean isValid(String s) {
        // stackとして使用する
        Deque<Character> stack = new ArrayDeque<>();
        Set<Character> openBrackets = new HashSet<>();
        openBrackets.add('(');
        openBrackets.add('[');
        openBrackets.add('{');

        for (Character character : s.toCharArray()) {
            if (openBrackets.contains(character)) {
                stack.push(character);
            } else if (character.equals(')')) {
                if (Objects.equals(stack.peek(), '(')) {
                    stack.pop();
                } else {
                    return false;
                }
            } else if (character.equals(']')) {
                if (Objects.equals(stack.peek(), '[')) {
                    stack.pop();
                } else {
                    return false;
                }
            } else if (character.equals('}')) {
                if (Objects.equals(stack.peek(), '{')) {
                    stack.pop();
                } else {
                    return false;
                }
            }
        }
        
        return stack.isEmpty();
    }
}
```
- 冗長な部分のコードを整理
```java
class Solution {
    public boolean isValid(String s) {
        // stackとして使用する。開き括弧を積む
        Deque<Character> stack = new ArrayDeque<>();
        // Map<開き括弧, 閉じ括弧>
        Map<Character, Character> brackets = new HashMap<>();
        brackets.put('(', ')');
        brackets.put('[', ']');
        brackets.put('{', '}');

        for (Character character : s.toCharArray()) {
            // 開き括弧の場合
            if (brackets.containsKey(character)) {
                stack.push(character);
                continue;
            } else /*閉じ括弧の場合*/ {
                if (stack.isEmpty()) {
                    return false;
                }
                // 直前の開き括弧と対応しない閉じ括弧ならfalse
                Character openBracket = stack.pop();
                Character closeBracket = brackets.get(openBracket);
                if (!Objects.equals(closeBracket, character)) {
                    return false;
                }
            }
        }
        return stack.isEmpty();
    }
}
```

# step2 他の方の解答を見る
- 参考
  - https://github.com/yamashita-ki/codingTest/pull/7
  - https://github.com/nanae772/leetcode-arai60/pull/7
## 解答
- 処理の流れは元のままでもいいかと思い、変数名を修正
  - Mapの命名でkeyToValueとするのが分かりやすいと思った
    - このMapをcloseToOpenとした方が処理は簡潔になる気がするが、どちらの方が分かりやすいだろうか
      - 個人的にはopenToCloseの方が分かりやすいような
```java
class Solution {
    public boolean isValid(String s) {
        Deque<Character> openBracketsStack = new ArrayDeque<>();
        // HashMap<開き括弧, 閉じ括弧>
        Map<Character, Character> openToClose = new HashMap<>();
        openToClose.put('(', ')');
        openToClose.put('[', ']');
        openToClose.put('{', '}');

        for (Character bracket : s.toCharArray()) {
            // 開き括弧の場合
            if (openToClose.containsKey(bracket)) {
                openBracketsStack.push(bracket);
                continue;
            } else /*閉じ括弧の場合*/ {
                if (openBracketsStack.isEmpty()) {
                    return false;
                }
                // 直前の開き括弧と対応しない閉じ括弧ならfalse
                Character openBracket = openBracketsStack.pop();
                Character closeBracket = openToClose.get(openBracket);
                if (!Objects.equals(bracket, closeBracket)) {
                    return false;
                }
            }
        }
        return openBracketsStack.isEmpty();
    }
}
```
- closeToOpenでも書いてみる
```java
import java.util.Objects;

class Solution {
    public boolean isValid(String s) {
        Deque<Character> openBracketsStack = new ArrayDeque<>();
        // HashMap<開き括弧, 閉じ括弧>
        Map<Character, Character> closeToOpen = new HashMap<>();
        closeToOpen.put(')', '(');
        closeToOpen.put(']', '[');
        closeToOpen.put('}', '{');

        for (Character c : s.toCharArray()) {
            // 閉じ括弧の場合
            if (closeToOpen.containsKey(c)) {
                if (openBracketsStack.isEmpty() || Objects.equals(openBracketsStack.peek(), closeToOpen.get(c))){
                    return false;
                }
                openBracketsStack.pop();
                continue;
            }
            // 開き括弧の場合
            openBracketsStack.push(c);
        }
        return openBracketsStack.isEmpty();
    }
}
```
- 再帰下降構文解析による実装もある
  - [nanae772さんの実装](https://github.com/nanae772/leetcode-arai60/pull/7/files#diff-fdd23252c7e768db41d11475cc4b0e799bd053196f897e350b6a823250a75e55) をそのままJavaにしたもの
  - 再帰的な関数呼び出しのコールスタックが、プッシュダウンオートマトンのスタックを表現しているということか
    - 開き括弧→再帰関数呼び出し(スタックに積む)→閉じ括弧を見つける→再帰関数終了(スタックから取り除く)
```java
class Solution {
    public boolean isValid(String s) {
        // 検証用パーサを生成
        BracketsStringValidator validator = new BracketsStringValidator(s);
        return validator.isValid();
    }
}

class BracketsStringValidator {
    private final String s;
    private int i = 0;

    public BracketsStringValidator(String s) {
        this.s = s;
    }

    public boolean isValid() {
        // 最後までパースでき、かつ入力をすべて消費したら true
        return canParse() && i == s.length();
    }

    private Character peek() {
        if (i >= s.length()) {
            return null;
        }
        return s.charAt(i);
    }

    private boolean consume(char ch) {
        if (!Character.valueOf(ch).equals(peek())) {
            return false;
        }
        i++;
        return true;
    }

    private boolean canParse() {
        // 文法:
        // S -> '(' S ')' | '[' S ']' | '{' S '}' | (空文字)
        java.util.Map<Character, Character> openToClose = new java.util.HashMap<>();
        openToClose.put('(', ')');
        openToClose.put('{', '}');
        openToClose.put('[', ']');

        Character next = peek();
        if (next == null || !openToClose.containsKey(next)) {
            // 空文字列または開き括弧以外ならOK
            return true;
        }

        char openBracket = next;
        char closeBracket = openToClose.get(openBracket);

        if (!(consume(openBracket) && canParse() && consume(closeBracket))) {
            return false;
        }
        return canParse();
    }
}

```

# step3 3回ミスなく書く
- 所要時間: 5分程度
- step2とほぼ同じだが、書き直してみたら、else文が一つ消えていた
## 解答
```java
class Solution {
    public boolean isValid(String s) {
        Deque<Character> openBracketsStack = new ArrayDeque<>();
        Map<Character, Character> openToClose = new HashMap<>();
        openToClose.put('(', ')');
        openToClose.put('[', ']');
        openToClose.put('{', '}');

        for (Character bracket : s.toCharArray()) {
            if (openToClose.contains(bracket)) {
                openBracketsStack.push(bracket);
                continue;
            }
            if (openBracketsStack.isEmpty()) {
                return false;
            }
            Character openBracket = openBracketsStack.pop();
            Character closeBracket = openToClose.get(openBracket);
            if (!Objects.equals(bracket, closeBracket)) {
                return false;
            }
        }
        return openBracketsStack.isEmpty();
    }
}
```
