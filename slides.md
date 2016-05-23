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

Refs:

- [Programming Pearls: A literate program](http://cecs.wright.edu/~pmateti/Courses/4180/Lectures/Examples/Knuth-CWP/cwp-knuth-cacm-1986.pdf)
- [Blog by Dr. Drang: More shell, less egg](http://www.leancrew.com/all-this/2011/12/more-shell-less-egg/)

---

### Goal of this presentation

- Know what bash CAN and CAN NOT do
- Know best practices that make scripting to be run
- Know traps and pitfalls you want to avoid
- Know where to get help

---

## Outline

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

### When to use shell scripting

Shell scripting is efficient when used to

- Do things like a "wrapper" - i.e. "shell"
- Use existing powerful Unix commands and tools
- Glue components together
- Install/start/stop scripts, system admin, automation

---

### When to use shell scripting (cont.)

Why bash?

- Bourne-Again SHell
- Standard shell installed on almost every distros
- Still growing (bash 4 v.s. ksh93)

---

### When NOT to use shell scripting

- Speed (performance) is a factor
- Extensive file operations required (lseek, flock, etc.)
- Need complex data structures
- Need direct access to system hardware
- Need socket I/O
- Too many arbitrary precision calculation

---

### Some programming common sense

Know your environment

- PATH, env, cwd, compatibility issues
- Don't run as root as possible as you can

Coding style

- Text width, indention, comments
- Naming conventions (src tree, variables, functions)
- No hard code, don't repeat

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

