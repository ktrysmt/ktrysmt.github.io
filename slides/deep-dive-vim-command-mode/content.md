## Whoami

<img style="float:left" class="avatar" data-src="https://avatars3.githubusercontent.com/u/6156742?v=3&s=460">

* Kotaro Yoshimatsu a.k.a ktrysmt

* 経歴
  * 学生時代 Perl monger
  * 受託開発会社 5年くらい
  * 2015年 リクルートにJOIN

---

## Scope

### Target

Who understand basic operations

--

### In Scope

- Understand combined KEY in Command Mode

--

### Out of Scope

- Hack of .vimrc
- Explain basic operation and behavior
- List plugins

---

## Command Mode ?

Also named Normal Mode

--

Can you understand this ?

--

yis

:open_mouth:

--

yf3e

:sleepy:

--

{c5}

:innocent:

---

## Command Mode maked up by two facts

1. TEXT OBJECT
2. OPERATOR

---

## 1. Text Object

---

## Text Object ?

= Any Rnage.

> The range of any selected character string

--

The Range is determined by

- Motion

Or,

- Visual Mode

---

## Motion ?

---

## Motion

Basic;

|MOTIONS|KEY|
|---|---|---|
|left-right-motions| {count} + h,l,^,$,0,f,F,t,T |
|up-down-motions| {count} + k,j,gk,gj,G,gg,% |
|word-motions| {count} + w,W,e,E,b,B,ge,gE,s,p |
|object-motions| {count} + (,),{,} |

--

Example;

|KEY|RANGE|
|---|---|
|5j| Five rows down |
|5w| Three words forward |
|4$| Four rows lower line end|


--

Also append that;

|MEAN|KEY|
|---|---|
|around|a|
|inner|i|

--

Example combined them;

|KEY|RANGE|
|---|---|
|aw | A word and whitespace behind it |
|iw| A word (exclude whitespace) |
|a"| Words enclosed in doublequotes (include quotes) |
|i"| Words enclosed in doublequotes (exclude quotes) |
|as| A Sentence and whitespace behind it |
|is| A Sentence (exclude whitespace) |

---

## 2. Operator

---

## Operator ?

|KEY|BEHAVIOR|
|---|---|
|y| Yank (Copy) |
|c| Change (Delete then Switch Insert Mode) |
|d| Delete |
|~| Toggle uppercase or lowercase |
|u| Make lowercase |
|U| Make uppercase |
|>| Indent right |
|<| Indent left |

---

## Combine Text Object and Operator

---

## Combine Text Object and Operator

|KEY|BEHAVIOR|
|---|---|
|yiw| Yank a word (exclude whitespace) |
|ya"| Yank words enclosed in doublequotes (include quotes) |
|d5j| Delete the Lower five rows |

---

## Conclusion

- Focus on Operator and Text Object
- Learn a combination easy-to-use


---


## thank you for your attention

### References

- [Vim のテキスト編集 〜基礎編〜 \- Vim のブログ](http://vimblog.hatenablog.com/entry/vim_edit_basic)
- [Vimのオペレーターとモーションについて ‹ 技術の犬小屋](http://promamo.com/?p=1968)
- [Vim のモーションについて \- Qiita](http://qiita.com/b4b4r07/items/7fc12842d03e7e46412c)




