# step1 何も見ずに解く
- 以下の要領で生成したアドレスをSetに保持
  - @の前後で分ける。
  - localNameを以下の要領で加工する
    - localNameの+以下の部分を切り捨てる
    - .を取り除く
  - localNameとdomainNameを連結する
- 正規表現一発で書けるのかもしれないが、加工の条件が後で増えたり減ったりしそうなので条件毎に分けて書きたい気持ちになる
- 問題の制約上必ず「@」が1つだけ含まれるが、未検証でインデックスアクセスするのが不安で一応ガードするようにした
## 解答

```java
class Solution {
  public int numUniqueEmails(String[] emails) {
    Set<String> uniqueEmails = new HashSet<>();
    // Stream APIで書くとこんな感じ
    // uniqueEmails.addAll(
    //      Arrays.stream(emails)
    //          .map(e -> canonicalizeEmail(e)).toList()
    // );
    for (String email : emails) {
      String canonicalizedEmail = canonicalizeEmail(email);
      if (canonicalizedEmail != null) {
        uniqueEmails.add(canonicalizedEmail);
      }
    }
    return uniqueEmails.size();
  }

  private String canonicalizeEmail(String email) {
    if (!email.contains("@") || email.indexOf("@") != email.lastIndexOf("@")) {
      // あるいはExceptionを投げる?
      return null;
    }

    String[] localNameAndDomainName = email.split("@");
    String localName = localNameAndDomainName[0];
    String domainName = localNameAndDomainName[1];

    // 「+」以下の部分を切り捨てる
    int firstPlusIndex = localName.indexOf("+");
    if (firstPlusIndex > -1) {
      localName = localName.substring(0, firstPlusIndex);
    }

    // 「.」を取り除く
    localName = localName.replaceAll("\\.", "");

    return localName + "@" + domainName;
  }
}
```

# step2 他の方の解答を見る
- 元の文字列を一文字ずつチェックして、正規化されたアドレスを生成する方法は思い浮かべていなかった
- step1の入力値チェックは以下のように書いた方が簡潔だった
  - `if (localNameAndDomainName.length != 2) return null;`
    - この「2」は何を表していると一瞬考えないといけない気もするので、現状の方が意図が伝わりやすいか?
- localNameから「+」以下を切り出すのは以下のように書く方が簡潔
  - `localName = localName.split("+")[0];`
- Javaの文字列テンプレートって書いたことないなと思って確認
  - JDK21でプレビュー公開されてたがJDK23で取り下げられてた
    - `STR."\{localName}@\{domainName}"`という具合にかけたらしい
    - https://nipafx.dev/inside-java-newscast-71/
## 解答
- 元の文字列を一文字ずつチェックして、正規化されたアドレスを生成する
- https://github.com/seal-azarashi/leetcode/pull/14/files
```java
class Solution {
    public int numUniqueEmails(String[] emails) {
        Set<String> uniqueEmails = new HashSet<>();
        for (String email : emails) {
            String canonicalizedEmail = canonicalizeEmail(email);
            if (canonicalizedEmail != null) {
                uniqueEmails.add(canonicalizedEmail);
            }
        }
        return uniqueEmails.size();
    }

    private String canonicalizeEmail(String email) {
        String[] localNameAndDomainName = email.split("@");
        // localNameとDomainNameの2つに分割できない場合
        if (localNameAndDomainName.length != 2) {
            return null;
        }
        String localName = localNameAndDomainName[0];
        String domainName = localNameAndDomainName[1];

        StringBuilder canonicalLocalName = new StringBuilder();
        for (char c : localName.toCharArray()) {
            if (c == '+') {
                break;
            }
            if (c == '.') {
                continue;
            }
            canonicalLocalName.append(c);
        }

        return canonicalLocalName.append("@").append(domainName).toString();
    }
}
```
- 正規表現を使った書き方
- Leetcodeってjava.util直下のクラスはimport不要だけど、java.util.regexとかはimport必要なのか
- https://github.com/nanae772/leetcode-arai60/pull/15/files
```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

class Solution {
  public int numUniqueEmails(String[] emails) {
    Set<String> uniqueEmails = new HashSet<>();
    for (String email : emails) {
      String canonicalizedEmail = canonicalizeEmail(email);
      if (canonicalizedEmail != null) {
        uniqueEmails.add(canonicalizedEmail);
      }
    }
    return uniqueEmails.size();
  }

  private String canonicalizeEmail(String email) {
      // 「ローカル名」「ローカル名の+以下の部分」「ドメイン名」の3領域をマッチ
    Pattern emailPattern = Pattern.compile("^([a-z\\.]+)(\\+[a-z\\.]*)?@([a-z\\.\\+]+)$");
    Matcher matcher = emailPattern.matcher(email);
    if (!matcher.matches()) {
      return null;
    }
    String localName = matcher.group(1);
    String domainName = matcher.group(3);
    localName = localName.replaceAll("\\.", "");
    return localName + "@" + domainName;
  }
}
```
# step3 3回ミスなく書く
## 解答
- Step1と同じ書き方で。面倒になってvalidationは書かず
```java
class Solution {
    public int numUniqueEmails(String[] emails) {
        Set<String> uniqueEmails = new HashSet<>();
        for (String email : emails) {
            uniqueEmails.add(canonicalizeEmail(email));
        }
        return uniqueEmails.size();
    }

    private String canonicalizeEmail(String email) {
        String[] localNameAndDomainName = email.split("@");
        String localName = localNameAndDomainName[0];
        String domainName = localNameAndDomainName[1];
        
        localName = localName.split("\\+")[0];
        localName = localName.replaceAll("\\.", "");
        return localName + "@" + domainName;
    }
}
```
