---
title: Tries
date: '2022-04-10'
categories: ["DSA"]
tags: ["Trie", "Python", "English Article"]
---

# Tries

It is commonly used to represent a dictionary for looking up words in a vocabulary

## Implementation

<https://leetcode.com/problems/implement-trie-prefix-tree/>

Trie() Initializes the trie object.
void insert(String word) Inserts the string word into the trie.
boolean search(String word) Returns true if the string word is in the trie (i.e., was inserted before), and false otherwise.
boolean startsWith(String prefix) Returns true if there is a previously inserted string word that has the prefix prefix, and false otherwise.

```python
class Trie:

    def __init__(self):
        self.root = TrieNode("")

    def insert(self, word: str) -> None:
        curr = self.root
        for c in word:
            if not c in curr.children: curr.children[c] = TrieNode(c)
            curr = curr.children[c]
        curr.is_word = True

    def search(self, word: str) -> bool:
        c = self.getNode(word)
        if c is not None and c.is_word: return True
        return False

    def startsWith(self, prefix: str) -> bool:
        c = self.getNode(prefix)
        if c is not None: return True
        return False

    def getNode(self, word: str):
        curr = self.root
        for c in word:
            if not c in curr.children: return None
            curr = curr.children[c]
        return curr

class TrieNode:
    def __init__(self, char: str):
        self.char = char
        self.is_word = False
        self.children = {}



# Your Trie object will be instantiated and called as such:
# obj = Trie()
# obj.insert(word)
# param_2 = obj.search(word)
# param_3 = obj.startsWith(prefix)
```

## Reference

[Implementing Trie in Python](https://albertauyeung.github.io/2020/06/15/python-trie.html/)
