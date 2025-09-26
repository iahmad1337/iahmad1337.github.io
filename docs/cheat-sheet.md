# Bash
```Bash
ag "message ProtoMessage" -G ".*\.proto$"

time soap make -i my_target && tmux display-popup -E sleep 1 && (cd testdir/my_target && ./test.sh &>/tmp/testlog; less /tmp/testlog)

curl -d private=0 -d name=ahmad1337 -d api_dev_key=$key -d api_paste_name="file.cpp" --data-urlencode text@/tmp/file.cpp https://paste228.com/api/v2/publish

svn diff -r r10:r22 --summarize ^/branches/ahmad1337/TICKET-228

# calculator
python3 -c "print(2**20)"

svn diff -r HEAD:PREV config.xml
svn diff -r HEAD:{2025-01-20} path/to/file.cpp

combinediff header.hpp.diff source.cpp.diff >>result.diff

# sort files by sizes accounting for suffixes (K for kilo, M for mega and etc.)
(for i in $(ls); do du -xhs $i; done) | sort --key=1 -h

# scp via jumphost-2 (is configured in ~/.ssh/config)
scp -F ~/.ssh/config -J jumphost-2 ahmad1337@my.machine.ru:/tmp/logs.tar.gz .

# Interpret bytes as x64 instructions
ndisasm -a -b64 machine-code-dump
# skip first 5 bytes
ndisasm -e 5 -b64
# skip the bytes in range [4, 7)
ndisasm -k 4,3 -b64

# compile to demangled asm (pre-linker)
g++ -std=c++17 -S -masm=intel src/01-qlibs-jmp.cc -fPIC -I./include -o - | c++filt >jmp.asm
# post-linker disasm
g++ -std=c++17 src/03* -fPIC -I./include -g -o 03.out && objdump --disassemble -Mintel 03.out | c++filt >03.asm

# get sections of and ELF file (+ their write/execute permissions)
readelf -S my-program

# check that function is indeed in the executable
nm my-test-executable | grep SpecificTest | c++filt

# List all files, opened by PID
lsof -p <PID>

# get environ of a process
strings /proc/<PID>/environ

# remove all invalid utf-8 sequences from file
iconv -f utf-8 -t utf-8 -c file.txt
```

# GDB
```
# launch gdb with some pre-run commands
gdb -x gdb-commands.txt html_idx

# print program output at terminal /dev/pts/15 (can be determined by writing tty
# in target terminal
gdb --tty=/dev/pts/15 -x /tmp/gdb-commands.txt --args ./binary-name --some-binary-flag --flag-2

fin (finish) - execute entire current frame and go back to parent frame
u (until) 145 - execute all commands up until (but not including) line 145

backtrace (bt) - print backtrace
bt full - backtrace with local variables
down, up - go up and down the backtrace
print <expression> - ochevidno

# source viewing
dir /home/ahmad1337/worktree/ - add one more lookup path for sources
show directories  - show the source lookup paths
list src/kek.cpp:42  - go to this location

# TUI
tui enable/disable - can be toggled with "ctrl-x + a"
```

Example debugger script with scrollback + output in separate terminals:
```bash
#!/usr/bin/env bash
cat << EOF >/tmp/gdb-script.txt
tui enable
tty /dev/pts/18
set trace-commands on
set logging on
EOF

pushd build
ninja && gdb -x /tmp/gdb-script.txt ./${BIN:-04-multithreaded}
popd
```
- `set trace-commands` on & `set logging on` sends all output of gdb commands to
  $WORK_DIR/gdb.txt, which can be viewed in another terminal pane using `tail -f gdb.txt`
- `tty /dev/pts/18` sends all program output to terminal `/dev/pts/18` (can be
  determined via `tty` shell command.

# network
```bash
# send one tcp request and wait for a response
head -n1 queries-concatenated.txt | nc --no-shutdown -4 8.8.8.8 8000

# Get IP of a hostname (single AA/AAAA record)
$ host 1.host.com                        # concrete instance (1 host)
1.host.com has address 10.20.6.2
# Name all AA/AAAA records behind this domain (in case of round-robin DNS)
$ host host.com                          # all instances on cloud HC
host.com has address 10.20.6.2
host.com has address 10.20.6.4
host.com has address 10.20.6.5
host.com has address 10.20.6.3
host.com has address 10.20.6.6

# List the listened-on ports and corresponding processes
ss -tlnp
```

# processes
```bash
# see all mappings of a process that take tens and hundreds of megabytes
grep -B4 -E "Rss:[[:blank:]]*[0-9]{5,6}" /proc/273/smaps

# Calculate honest RSS of a process that contains "worker" in its full command line
cat /proc/$(pgrep --full --oldest worker)/smaps | grep Rss | awk '{s += $2}END{print s/1024 " MiB"}' 2>/dev/null | grep MiB

# show all processes and the their entire command line
ps all -e ww

# kill process by name
pkill --signal SIGKILL "udp_shooter"  # use -f for full process command match

# show memory map sizes of a process
pmap $(pgrep --full --oldest worker) | sort -n -k2

# Show all (-e) processes + threads (-T) + full command (-f). Ps doesn't show separate threads by default
ps -efT

# Show all threads named "worker" for user ahamd1337
ps -u ahmad1337 -fT | grep worker
```

# system
```bash
# check, if kernel was compiled with CONFIG_FTRACE option (/boot/config-* may also be in /proc/config.gz)
for i in /boot/config-*; do zgrep "CONFIG_FTRACE" "$i"; done
```

# vim
```vim
:%!c++filt  -- demangle mangled identifiers in file (works for disasm or any
            -- other human-readable file)

:g/worker-1[0-9]/d  -- remove lines matching the pattern
```

# git
```bash
# status without untracked files
git status -uno
```
