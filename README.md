# Chinese aiml robots implement

use jieba and python2 aiml packages

## 1. how to use

1. need python2 environment

2. use pip command to install dependencies

   ```shell
   pip install textrank4zh
   ```

   run main.py to for dialogue

   ```shell
   python main.py
   ```

### 1.1 deploy webpage 

dependencies need to be installed if deploy webpage server

```shell
pip install web.py
```

   then run application program

   ```shell
   python run application.py
   ```

   access the webpage 'localhost:8080'


## 2. write AIML

### 2.1 Custom thesaurus

Enter a custom vocabulary in the file `source/user-dict` to avoid incorrect word segmentation:

```
春节联欢晚会 10 n
```

In the original stuttering participle, the word and part-of-speech tagging of the custom lexicon are not required, but in the implementation of the textrank4zh package, the part-of-speech tagging is required to be used correctly.

For part-of-speech tagging, see: [Word of Speech](https://gist.github.com/luw2007/6016931). The most commonly used part of speech is a noun, labeled "n".

*Note: Do not save the custom dictionary with Windows Notepad*

### 2.2 Test keyword extraction results

Use split-test.py to test the word segmentation and extract the results to determine what to write in the pattern tag.

Use the custom thesaurus above to avoid some errors; type `reload` when split-test is running to update the word breaker dictionary in real time. (Note that you need to execute the command after saving the changed user-dict file)

### 2.3 Synonym replacement

Remove DefaultSubs.py and WordSub.py from the original aiml application and add the file Sub.py to achieve the function.

The words that need to be replaced are stored in `source/subs` with the format of `original-word substitution-word`.

For example, when input: "我喜欢踢球", we can extract the keywords "我 喜欢 踢球". According to the rules in the replacement dictionary [喜欢 爱], replaced with "我 爱 踢球". When matching with "我 爱 踢球" pattern rule, we could eliminate the need for extra pattern writing.

Because of the reduced size of the Trie tree, this alternative vocabulary strategy should be better than the following 'simplified writing' considerations. 

But we must add replaceable vocabulary for synonyms that can be substituted for each other to avoid ambiguity and subsequent errors.

### 2.4 Simplified writing

Added the unfold_shorts.py file, which will assist in parsing the pattern content to partially simplify the writing of the aiml file.

Take the file "test.aiml" as an example:

```xml
<category>
  <pattern> I
    <fold>
      <or>LIKE</or>
      <or>HATE</or>
    </fold> YOU
  </pattern>
  <template>balabala</template>
</category>
```

Analytic generation test-unfold.aiml

```xml
<category>
  <pattern> I LIKE YOU </pattern>
  <template>balabala</template>
</category>
<category>
  <pattern> I HATE YOU </pattern>
  <template>balabala</template>
</category>
```

For the ‘fold’ tag in the pattern, expand the ‘or’ tag content in it, and splicing with the original pattern text content to generate multiple categories.

If you use unfold_shorts.py to parse the original aiml file, you need to specify the target file in `unfdld_list` and the `-unfold.aiml` suffix** in the `learn` tag of the `startup.xml` file.

> Note: English content in the pattern tag needs to be all uppercase

### 2.5 star Usage change

original aiml implementation, the wildcard `*` indicates a match to any word. After modification, the current `*` will match 0 or more words.

Correspondingly, when the input is obtained using the star tag, an empty string (no corresponding input), a correctly matched word, and multiple words separated by spaces (multiple inputs matching the rule) are returned.

Since the match for normal words in the program is before the wildcard, some tricks can be used.
For example, the following aiml content:

```xml
  <category>
    <pattern>什么 是</pattern>
    <template>什么是什么呢？</template>
  </category>
  <category>
    <pattern>* 什么 是 *</pattern>
    <template>我们会为你查询 <star index="2"/></template>
  </category>
```

The dialogue is as follow:

```
user：什么是
bot：什么是什么呢？
user：什么是足球
bot：我们会为你查询 足球
```

