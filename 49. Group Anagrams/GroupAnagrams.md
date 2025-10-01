# step1 何も見ずに解く
- ナイーブな実装だととても時間がかかりそう。効率的に処理する方法を考える
  - HashMap<アルファベット順にソートしたstr, List<元のstr>>を作っていくのがいいか
  - それぞれの単語の構成文字の出現個数をカウントする方法も思い浮かんだが、それをうまくHashMapのキーに変換する方法がパッと出てこなかった
    - 1度Mapにつめて、ソートしながら連結するみたいなことを考えていた
      - これであれば最初からソートしたほうがシンプルではやい
    - step2で見てたら、入力がアルファベットのみであることを利用すればよかった
## 解答
```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String, List<String>> sortedToAnagrams = new HashMap();
        for (String str : strs) {
            String sortedStr = sortString(str);
            sortedToAnagrams
                    // 新規登録の場合は初期値として空の配列をセット
                    .computeIfAbsent(sortedStr, (key) -> new ArrayList<>())
                    .add(str);
        }
        return new ArrayList<>(sortedToAnagrams.values());
    }

    private String sortString(String unsorted) {
        char[] chars = unsorted.toCharArray();
        Arrays.sort(chars);
        return new String(chars);
    }
}
```

# step2 他の方の解答を見る
- ソート文字列をキーにする、構成文字カウントをキーにする以外の実装は見当たらず
## 解答
- https://github.com/Shinkomori19/arai60/pull/3/files
- 構成文字の出現個数をカウントしたものをキーにする実装
```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String, List<String>> anagramKeyToAnagrams = new HashMap();
        for (String str : strs) {
            String anagramKey = genAnagramKey(str);
            anagramKeyToAnagrams
                    // 新規登録の場合は初期値として空の配列を登録
                    .computeIfAbsent(anagramKey, (key) -> new ArrayList<>())
                    .add(str);
        }
        return new ArrayList<>(anagramKeyToAnagrams.values());
    }

    private String genAnagramKey(String unsorted) {
        char[] chars = unsorted.toCharArray();
        int[] alphabetCount = new int[26];
        for (char c: chars) {
            //「-'a'」すると
            // a → 0, b → 1, z → 26という具合になる
            alphabetCount[c - 'a']++;
        }
        StringBuilder sb = new StringBuilder();
        for (int count : alphabetCount) {
            sb.append("#").append(count);
        }
        return sb.toString();
    }
}
```
- 今回の入力では考慮不要だがサロゲートペアに配慮した実装も試してみる。これで絵文字や変な漢字がきても処理できる。
```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String, List<String>> sortedToAnagrams = new HashMap<>();
        for (String str : strs) {
            String sorted = sortStringWithCodePoint(str);
            sortedToAnagrams
                    .computeIfAbsent(sorted, (k) -> new ArrayList<>())
                    .add(str);
        }
        return new ArrayList<>(sortedToAnagrams.values());
    }

    private String sortStringWithCodePoint(String str) {
        int[] codePoints = str.codePoints().toArray();
        Arrays.sort(codePoints);
        StringBuilder sb = new StringBuilder();
        for (int codePoint : codePoints) {
            sb.appendCodePoint(codePoint);
        }
        return sb.toString();
    }
}
```
# step3 3回ミスなく書く

## 解答
- step1と同様の実装
```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> sortedToAnagrams = new HashMap<>();
        for (String str : strs) {
            String sorted = sortString(str);
            sortedToAnagrams
                .computeIfAbsent(sorted, (key -> new ArrayList<>()))
                .add(str);
        }
        return new ArrayList<>(sortedToAnagrams.values());
    }

    private String sortString(String str) {
        char[] chars = str.toCharArray();
        Arrays.sort(chars);
        return new String(chars);
    }
}
```
