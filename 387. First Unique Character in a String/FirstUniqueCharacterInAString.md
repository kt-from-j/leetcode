# step1 何も見ずに解く
- 各文字について、初登場のインデックス・登場回数を知りたい
  - Keyは文字としてMapのvalueとして{firstFoundIndex, foundCount}を保持すればいいのだが、クラスを宣言するほどでもないかと思いやめた
## 解答
- Mapを2つ使用した実装
  - `charToFirstFoundIndex`無しで`s.indexOf()`やsの要素に対するforループで十分だったかも
```java
class Solution {
    public int firstUniqChar(String s) {
        // Map<文字、登場回数>
        Map<Character, Integer> charToFoundCount = new LinkedHashMap<>();
        // Map<文字、初登場のインデックス>
        Map<Character, Integer> charToFirstFoundIndex = new HashMap<>();

        char[] chars = s.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            char c = chars[i];
            charToFoundCount.merge(c, 1, Integer::sum);
            if (!charToFirstFoundIndex.containsKey(c)) {
                charToFirstFoundIndex.put(c, i);
            }
        }

        for (Map.Entry<Character, Integer> entry : charToFoundCount.entrySet()) {
            if (entry.getValue() == 1) {
                return charToFirstFoundIndex.get(entry.getKey());
            }
        }
        return -1;
    }
}
```
- Set1つの実装
  - forループは一つだけど、lastIndexOfしているから最悪計算量はO(n^2)になるのかな
```java
class Solution {
    public int firstUniqChar(String s) {
        char[] chars = s.toCharArray();
        Set<Character> foundChar = new HashSet<>();
        for (int i = 0; i < chars.length; i++) {
            char c = chars[i];
            if (i == s.lastIndexOf(c) && !foundChar.contains(c)) {
                return i;
            }
            foundChar.add(c);
        }
        return -1;
    }
}
```

# step2 他の方の解答を見る
## 解答
- https://github.com/jeymak-trainee/arai60training/pull/4
- サロゲートペアに対応した実装
  - 次の文字に移る際に`i += Character.charCount(codePoint);`としなければいけない点に注意が必要
    - この実装って入力が"😀a😀"みたいな場合に2を返すことになるが、それでいいのかは要相談かも
      - 見た目上は2文字目なので1を返す方が多くの場合、自然に思われる。
```java
class Solution {
    public int firstUniqChar(String s) {
        // Map<文字、登場回数>
        Map<Integer, Integer> charToFoundCount = new LinkedHashMap<>();
        for (int i = 0; i < s.length();) {
            int codePoint = s.codePointAt(i);
            charToFoundCount.merge(codePoint, 1, Integer::sum);
            i += Character.charCount(codePoint);
        }

        for (int i = 0; i < s.length();) {
            int codePoint = s.codePointAt(i);
            if (charToFoundCount.get(codePoint) == 1) {
                return i;
            }
            i += Character.charCount(codePoint);
        }
        
        return -1;
    }
}
```

# step3 3回ミスなく書く
- 繰り返し書くと面倒になり、Step1をシンプルにした実装で書いた
## 解答
```java
class Solution {
    public int firstUniqChar(String s) {
        Map<Character, Integer> charToFoundCount = new HashMap<>();
        for (Character c : s.toCharArray()) {
            charToFoundCount.merge(c, 1, Integer::sum);
        }
        for (int i = 0; i < s.length(); i++) {
            Character c = s.charAt(i);
            if (charToFoundCount.get(c) == 1) {
                return i;
            }
        }
        return -1;
    }
}
```
