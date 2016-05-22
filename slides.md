class: center, middle

# Bash Scripting Best Practices

<img src="img/bash.png" />

Matthew Wang

<small>
Last updated: May 22, 2016<br>
[github.com/ymattw/bash-scripting-best-practices](https://github.com/ymattw/bash-scripting-best-practices)
</small>

---

class: center, middle

## Preface

A funny story

---

### Donald Knuth

<img class="right" src="img/knuth.gif" />

- Professor Emeritus of Computer Science at Stanford
- Author of _The Art of Computer Programming_
- "Father of algorithmic analysis"
- Creator of TeX

---

### The task

In 1986, Knuth was asked to write a guest feature for the "Programming Pearls"
column in the Communications of the _ACM_ journal.

The task was to write a program that would:

> Read a text file, determine the _n_ most frequently used words, and print out
> a sorted list of those words along with their frequencies.

---

### Knuth's solution

Knuth produced a solution in Pascal.  It was well designed, thoroughly
commented and used a novel data structure for managing the word count list.

It was _**10 pages**_ when printed.

---

### Douglas McIlroy

<img class="right" src="img/McIlroy.jpg" />

- Cornell University, MIT, Bell Laboratories, Oxford University
- Author of Unix pipelines and a bunch of Unix tools such as spell, diff, sort
  and tr, etc.

In response, Douglas McIlroy wrote a shell script that produced the same
output, but was only _**6 lines**_ long.

---

### Douglas McIlroy's shell script

```bash
tr -cs A-Za-z '\n' |
tr A-Z a-z |
sort |
uniq -c |
sort -rn |
sed ${1}q
```

---

### Goal of this presentation

- Know what bash CAN and CAN NOT do
- Know best practices that make scripting to be run
- Know traps and pitfalls you want to avoid
- Know where to get help

---

# Outline

- Common sense
- Principles
- Traps
- Debugging
- References

---

class: center, middle

## Common sense

When to use / not use bash scripting

---

class: center, middle

## Principles

Details of best practices

---

class: center, middle

## Traps

Pitfalls to know

---

class: center, middle

## Debugging

Troubleshooting techniques

---

## References

- http://www.leancrew.com/all-this/2011/12/more-shell-less-egg/
