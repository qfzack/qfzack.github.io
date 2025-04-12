---
title: "数据结构：字典(Trie)树的实现(Java)"

date: 2020-03-28T05:00:00Z
# image: "/images/placeholder.png"
categories: ["Data Structure"]
author: "Qingfeng Zhang"
---

字典(Trie)树又称作单词查找树，是哈希树的变种，应用于统计、排序和保存大量的字符串，常被用于搜索引擎的文本词频统计；字典树可以利用字符串的公共前缀来减少查询的时间；  
![](images/TrieTree/1.jpg)  
先定义**TrieTree的节点类** ，包括节点的**构造方法** 和对指定节点的字节点进行查找的**subNode()方法** ：

```java
class Node {  
    char val;  //节点内容  
    boolean isEnd;  //该节点能否形成单词  
    int count;  //该节点被几个单词共享  
    List<Node> childs;  //子节点的集合  
    
    //Trie树的初始化  
    public Node(char c){  
        val = c;  
        isEnd = false;  
        count = 0;  
        childs = new LinkedList<>();  
    }  
    
    //查找当前节点的子节点中是否包含c  
    public Node subNode(char c){  
        if(childs.size()==0){  
            return null;  
        }  
        for(Node node: childs){  
            if(node.val==c){  
                return node;  
            }  
        }  
        return null;  
    }  
}  
```
  
接下来定义**TrieTree类** ，包括树的**构造方法** ，向字典树中插入一个单词的**insert()方法**
，查找字典树中是否包含指定单词的**search()方法** ，从字典树中删除一个指定单词的**deleteWord()方法**
，输出当前所有单词的**printAll()方法** ：

```java
import java.util.*;  
    
public class NO820_trieTree {  
    private Node root;  
    
    //类的构造方法  
    public NO820_trieTree(){  
        root = new Node(' ');  
    }  
    
    //单词的插入  
    public void insert(String word){  
        if(search(word)) return ;  //单词已存在  
    
        Node cur = root;  
        for(char c: word.toCharArray()){  
            Node child = cur.subNode(c);  //在cur的子节点中查找当前字符  
            if(child!=null){  //子节点中包含当前字符  
                cur = child;  //更新cur指向子节点  
            }else{  
                cur.childs.add(new Node(c));  //在cur的子节点中添加当前字符  
                cur = cur.subNode(c);  //更新cur指向子节点  
            }  
            cur.count++;  //当前节点的共享单词数+1  
        }  
        cur.isEnd = true;  //最后一个节点是单词的结尾  
    }  
    
    //判断单词是否存在  
    public boolean search(String word){  
        Node cur = root;  
    
        for(char c: word.toCharArray()){  
            if(cur.subNode(c)==null){  //当前字符不存在trie中  
                return false;  
            }else{  
                cur = cur.subNode(c);  //更新cur节点  
            }  
        }  
        if(cur.isEnd){  //最后一个节点是不是单词的结尾  
            return true;  
        }else{  
            return true;  
        }  
    }  
    
    //单词的删除  
    public boolean deleteWord(String word){  
        if(!search(word)) return false;  //单词不存在  
    
        Node cur = root;  
        for(char c: word.toCharArray()){  
            Node child = cur.subNode(c);  //在子节点中查找当前字符  
            if(child.count==1) {  //如果当前节点只存在与word，直接删除节点就行  
                cur.childs.remove(child);  
                return true;  
            }else{  //否则移到当前字符在子节点中的位置  
                cur.count--;  //共享当前节点的单词数减一  
                cur = child;  
            }  
        }  
        cur.isEnd = false;  //当前的节点不再是单词的结尾  
        return true;  
    }  
    
    //打印trie中所有的单词  
    public void printAll(){  
        dfs(root,"");  
    }  
    private void dfs(Node node,String s){  
        if(node==null) return ;  
        if(node.childs.size()==0){  
            System.out.println(s);  
            return ;  
        }  
        for(Node n: node.childs){  
            dfs(n,s+n.val);  
        }  
    }  
    
    public static void main(String[] args){  
        NO820_trieTree trie = new NO820_trieTree();  
        String name1 = "zhangqingfeng";  
        String name2 = "zhangqf";  
        String name3 = "zackqf";  
    
        trie.insert(name1);  
        trie.insert(name2);  
        trie.insert(name3);  
        trie.printAll();  
    
        trie.deleteWord(name2);  
        System.out.println(trie.search(name1));  
        System.out.println(trie.search(name2));  
        trie.printAll();  
    }  
}  
```

运行结果为：

```
zhangqingfeng  
zhangqf  
zackqf  
true  
false  
zhangqingfeng  
zackqf  
```
