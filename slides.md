class: center, middle

# Bash Scripting Best Practices

<img src="img/bash.png" />

Matthew Wang

<small>
Last updated: May 23, 2016<br>
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
- Know pitfalls you want to avoid
- Know where to get help

---

## Outline

- Common sense
- Principles
- Pitfalls
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
- It can be very long (Google suggests less than 200 lines)

---

### Some programming common sense

Know your environment

- PATH, env, cwd, compatibility issues
- Don't run as root as possible as you can

Coding style

- Text width, indention, comments
- Naming conventions (src tree, variables, functions)
- No hard code, don't repeat
- Keep good readability

---

class: center, middle

## Principles

Details of best practices

---

### Organize source tree

Bad habit v.s. good habit

```
.                           .
|-- run.sh                  |-- bin/
|-- report.sh               |   |-- run.sh
|-- log.sh                  |   `-- report.sh
|-- util.sh                 |-- lib/
|-- report.sh               |   |-- liblog.sh
|-- mail.sh                 |   |-- libutil.sh
|-- ssh.sh                  |   |-- libreport.sh
|-- testconfig.sample       |   |-- libmail.sh
`-- logs/                   |   `-- libssh.sh
                            |-- conf/
                            |   `-- testconfig.sample
                            `-- logs/
```

---

### Naming conventions

Bad style v.s. good style

```
DEBUG_LVL=0                   # In liblog.sh
function info                 LOG_LEVEL=0
function error                function log_info
                              function log_error
function dump_variables
function ping_host            # In libutil.sh
                              function util_dump_variables
function setup_ssh            function util_ping_host
function remote_run
                              # In libssh.sh
                              function ssh_setup
                              function ssh_run
```

_**Prefix**_ is your buddy

---

### Namespcae

Sorry, no namespace available, use functions!

- Choose global names carefully, use uppercased letters
- Keep only global variables and the call to function _**`main`**_ outside
  functions
- Use _**`readonly`**_ whenever you can
- Use _**`local`**_ modifier for all function scope variables

```bash
readonly LOGBOOK_API_HOME="/export/servers/logbook-api"

function logbook_install
{
    local installer=${1:?}
    # ...
}

function main
{
    # ...
}

main "$@"
```

---

### Variable expansion

```bash
FOO=$BAR             # Or FOO=${BAR}
FOO=${BAR}_COUNT     # Now {} is mandatory
QUZ=${FOO}-${BAR}    # $FOO-$BAR works but with bad readability
NAME=$(id -u)        # Sub shell expansion
OUTPUT=$(myfunc)     # Note grabs stdout only
TMP=~/tmp/           # Expands to "$HOME/tmp"
TMP=~user/tmp        # Expands to user's $HOME/tmp
```

---

### Here document

Variables expands in here document

```bash
cat << EOF
Usage:
    $0 –a <adserver> -b <broker>
Example:
    $0 –a af1033 –b af1022
EOF
```

Avoid expanding in here document

- Escape: `\$` (and <code>\&#x60;</code>)
- `cat << \EOF`
- `cat << "EOF"`

---

### Wildcards

Wildcard is handy

- `*, ?, [a-z], /path/to/{foo,bar*}.conf`

But be aware

- `ssh $remotehost ls /opt/servers/logs/*.log`
- Solution: use quotes

Watch out, especially when use with "rm"!

- Pro tip: <span class="red">when type "rm" from command line, always review before press ENTER!</span>

---

### Wildcards (cont.)

<blockquote>
<pre>
Date: Wed, 10 Jan 90
From: djones@megatest.uucp (Dave Jones)
Subject: rm *
Newsgroups: alt.folklore.computers2

Anybody else ever intend to type:
   % rm *.o

And type this by accident:
   % rm *>o

Now you've got one new empty file called "o", but
plenty of room for it!
</pre>
</blockquote>

Ref: page 21 of [The Unix-haters handbook](http://simson.net/ref/ugh.pdf)
(Translations available on the web)

---

### Wildcards (cont.)

A more painful example (Oops, I am root!)

- <code>rm –rf $SERV<span class="red">RE</span>\_DIR/etc/\*</code>

Good habits

- Don't run as root, use `sudo` when you need root access
- `set -o nounset (set -u)`
- <code>rm –rf ${SERVER_DIR<span class="red">:?</span>}/etc/\*</code>

---

### Double quotes

Double quotes are always good

```bash
echo "Hello $USER, welcome to $(hostname -s)"
XML="<user id=\"$id\">"
ssh example.org "ls /opt/servers/logs/*.log"
echo "[INFO] $LINE"             # Keep leading/trailing blanks
echo "$(tail logfile)"          # Multi-line
curl "${cgi}?q=foo&id=123"      # Meta char
```

Unless you don't want to keep original output

```bash
NAME=$(echo $NAME)              # Blanks trimmed
```

---

### Single quotes

Avoid variables expanding

```bash
echo 'Total cost is $100'
awk '{print $2}' file.txt
```

Save escaping for double quotes

```bash
XML='<user id="123">'     # Comparing to: XML="<user id=\"123\">"
```

---

### Tests

Use `[[ .. ]]` not `test` or `[ .. ]`, here's why

```bash
test -f $NONEXIST; echo $?      # You get 0, oops
[ -f $NONEXIST ]; echo $?       # You get 0, oops
[[ -f $NONEXIST ]]; echo $?     # You get 1
test -n $NOTHISVAE; echo $?     # You get 0, oops
[ -n $NOTHISVAE ]; echo $?      # You get 0, oops
[[ -n $NOTHISVR ]]; echo $?     # You get 1
```

Arithmetic comparison: use `(( .. ))` not `[[ .. ]]`, the former does type
conversion for you

```bash
(( uid == 0 ))    # compare to (( $uid == 0 ))
```

---

### Function

Function name, use which one?

- `function foo`
- `foo()`
- `function foo()`

They all work, but my favorite one is `function foo`

- Easier to extract functions by `grep '^function' foo.sh`

---

### Function (cont.)

Parameters & local variables

```bash
local id=${1:?}                 # Required argument
local id=${1:?"id required"}    # Even better
local name=${2:-$USER}          # Optional, with default
local -i count=0                # Specify a type
```

Return code

- Return code is the last command executed, or
- Explicitly return _**0 for success**_, others for failure

---

### Exit code

Only `exit` from main function.

Be aware of $?

- It is the return code of most recent expression

Check sensitive exit code

- `do_something || return 1`

set -o errexit (set -e)

```bash
grep error $log && echo "failed"    # $? can be non zero
! grep error $log || echo "failed"  # Assertion
do_cleanup || true                  # Ignore failure
do_something | tee $log             # You need ${PIPESTATUS[0]}
```

Your script also needs to set exit code for caller

---

### Redirecting and piping

Redirecting and piping examples

```bash
: > file                # Or just "> file"
foo > log 2>&1          # Redirect both stdout & stderr
foo >& log              # Same as above, non POSIX
foo 2>&1 | grep -i ERROR >> error.log

while read LINE; do
    process "$LINE"
done < textfile

ssh $host tar –C /opt/servers -czf – logs | tar -C /tmp -xzvf -
```

---

### Compound expressions

Use `( ... )` in case you need to chdir

```bash
(
    echo "Log file is $LOG"
    cat $LOG
    cd $LOG_DIR
    generate_report "$LOG" > $LOG.html
    echo "$(date) Completed."
) | mail -s "Test Report" $MAIL_ADDR

```

Otherwise use `{ ... }` to save a fork (vfork)

```bash
(( $# == 2 )) || { usage; exit 1; }
```

---

### Signal trapping

Typical usage

```bash
trap "singal_hdl" INT           # Special action for ctrl-c

trap "" INT QUIT                # Ignore ctrl-c and ctrl-\

trap "rm -f $TMPDIR" EXIT       # script 'exit' handler

trap "rm -f $TMPFILE" RETURN    # function 'return' handler
```

---

class: center, middle

## Pitfalls

Traps to know

---

### set -o errexit (set -e)

Declare `set -o errexit` to fail early (another good option is `set -o
nounset`)

From bash(1):

```
-e  Exit immediately if a simple command (see  SHELL  GRAMMAR
    above)  exits with a non-zero status.  The shell does not
    exit if the command that fails is  part  of  the  command
    list immediately following a while or until keyword, part
    of the test in an if statement, part of a && or ||  list,
    or if the command's return value is being inverted via !.
    A trap on ERR, if  set,  is  executed  before  the  shell
    exits.
```

Must explicitly return/exit for these cases

- while, until, if
- `&&` or `||`
- `! foo`

Example: change to `! foo || return 1`

---

### Piping creates a sub process!

```bash
echo "$line" | read NAME ID
echo "$NAME - $ID"              # Oops! Got empty strings
```

Solution

```
read NAME ID <<< "$line"
```

---

### Subshell assignment

Note the return value

```bash
local output=$(foo)     # $? is always 0

local output
output=$(foo)           # should break into two expressions
```

---

### Shift

If number of args is less than 2, `shift 2` won't shift anything (might cause
infinite loop).

Use `shift; shift` instead.

---

class: center, middle

## Debugging

Troubleshooting techniques

---

### Debugging techniques

Syntax check with `bash -n script.sh`

Debug flag `set -x` (bash -x script.sh)

Print error to stderr, remember it might be called as `output=$(foo)`

```bash
echo "ERROR: ..." >&2
```

Don't drop diagnostic messages

```bash
count=$(foo | grep -c "$pattern")   # output from foo disappeared

# You might need:
#
output=$(foo) || {
    echo "ERROR: foo failed" >&2
    echo "$output" >&2
    return 1
}

count=$(echo "$output" | grep -c "$pattern") || true
```

---

## References

- Man page bash(1)
- [Bash reference manual](http://www.gnu.org/software/bash/manual/bashref.html)
- [Advanced Bash-Scripting Guide](http://tldp.org/LDP/abs/html/index.html)
- [Shell style guide](https://github.com/ymattw/shell-style-guide/blob/master/shell-style-guide-cn.md)

---

class: center, middle

## Thank you!

Matthew Wang<br>
[github.com/ymattw](https://github.com/ymattw)
