[1、LeetCode 299 猜数字游戏](https://leetcode-cn.com/problems/bulls-and-cows/submissions/)

思路：先遍历一次secret，找到guess和secret位置相同数字相同的进行统计，并且使用HashMap记录不相同数字的出现次数。然后再遍历guess，排除相同位置的数字，然后统计不同位置的相同数字的出现次数。

```java
class Solution {
    public String getHint(String secret, String guess) {
        int A = 0, B = 0;
        StringBuilder sb = new StringBuilder();
        HashMap<Character,Integer> map = new HashMap<>();
        for (int i = 0; i < secret.length(); i++) {
            if (secret.charAt(i) == guess.charAt(i)){
                A++;
            }else {
                map.put(secret.charAt(i), map.getOrDefault(secret.charAt(i),0) + 1);
            }
        }
        for (int i = 0; i < guess.length(); i++) {
            if (map.containsKey(guess.charAt(i)) && secret.charAt(i) != guess.charAt(i)){
                if (map.get(guess.charAt(i)) > 0) {
                    map.put(guess.charAt(i), map.get(guess.charAt(i)) - 1);
                    B++;
                }
            }
        }
        return sb.append(A).append("A").append(B).append("B").toString();
    }
}
```

简洁版：

```java
class Solution {
    public String getHint(String secret, String guess) {
        int[] nums = new int[10];
        int A = 0,B = 0;
        for (int i = 0; i < secret.length(); i++) {
            if (secret.charAt(i) == guess.charAt(i)){
                A++;
            }else {
                nums[secret.charAt(i) - '0']++;
            }
        }
        for (int i = 0; i < guess.length(); i++) {
            if (nums[guess.charAt(i) - '0'] > 0 && secret.charAt(i) != guess.charAt(i)){
                nums[guess.charAt(i) - '0']--;
                B++;
            }
        }
        return A + "A" + B + "B";
    }
}
```

2、[LeetCode 676 实现一个魔法字典](https://leetcode-cn.com/problems/implement-magic-dictionary/)

**如果一个单词中只有一个字符可以更改以使字符串相等，那么两个单词就是邻居。**

思路：首先针对字典中的每个单词，构造其所有的邻居，并且使用map存储每个邻居出现的次数，然后将所有单词放入set中。当进行搜索时，先构造搜索单词word所对应的邻居，然后从map中进行查询，当前构造的邻居为sb，如果`map.get(sb)>1 || map.get(sb)==1&&!set.contains(word)`返回`true`

当map.get(sb)>1，说明字典中存在两个以上的单词，它们只有一个位置上的字符不相同。下面列出几种情况：

dict：["hello","hallo"]      search:["hello"],["hhllo"]         返回两个true

dict:["hello"]  					search:["hello"]       				 返回false

dict:["hello"]      				search:["hallo"]						返回true

```java
class MagicDictionary {

    HashMap<String,Integer> map;
    Set<String> set;
    /** Initialize your data structure here. */
    public MagicDictionary() {
        map = new HashMap<>();
        set = new HashSet<>();
    }
    
    /** Build a dictionary through a list of words */
    public void buildDict(String[] dict) {
        for (String s : dict){
            for (int i = 0; i < s.length(); i++) {
                StringBuilder sb = new StringBuilder(s);
                sb.setCharAt(i,'*');
                map.put(sb.toString(),map.getOrDefault(sb.toString(),0) + 1);
            }
            set.add(s);
        }
    }
    
    /** Returns if there is any word in the trie that equals to the given word after modifying exactly one character */
    public boolean search(String word) {
        for (int i = 0; i < word.length(); i++) {
            StringBuilder sb = new StringBuilder(word);
            sb.setCharAt(i,'*');
            int c = map.getOrDefault(sb.toString(),0);
            if (c > 1 || c == 1 && !set.contains(word)){
                return true;
            }
        }
        return false;
    }
}

/**
 * Your MagicDictionary object will be instantiated and called as such:
 * MagicDictionary obj = new MagicDictionary();
 * obj.buildDict(dict);
 * boolean param_2 = obj.search(word);
 */
```
