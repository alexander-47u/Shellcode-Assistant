# Shellcode Assistant

A useful script for hex editing, writing, scanning and/or testing **Linux x86 shellcode** to bypass IDS/IPS.

**Tested on Kali Linux Rolling x86 & x64**

The script was developed for a Offensive Technologies research project by students of OS3. We were looking for a comfortable and efficient way of altering existing shellcode. This tool will help reproduce the experiments of our research.

It is designed to make the process of altering shellcode to bypass **signature based** and heuristic detection easier. It takes a raw shellcode file as input and offers various options, including scanning shellcode using YARA rules, dumping code in hexadecimal or Intel x86 ASM format, compiling code using a C wrapper and executing and checking a raw shellcode file for a specific string. In addition, it can emulate, analyse and generate a graph for x86 shellcode using Libemu. Automatic recompilation, which triggers when the modification time stamp changes, is another useful feature that might save time when raw shellcode is manually modified and saved using a hex editor.

**We recommend using the automatic recompilation mode by using the -R flag**.

The script also provides an **interactive command line interface**, similar to Metasploit's nasm-shell, for assembling and disassembling instructions. It does this using the Capstone and Keystone libraries.

## Dependencies
* Python 3 (≥ 3.5)
* Capstone
* Keystone
* YARA
* Colorama
* Libemu
* gcc

## Setup

```
# python3 libraries
pip3 install capstone
pip3 install --no-binary keystone-engine keystone-engine
pip3 install colorama
pip3 install yara

# gcc is required for compiling the raw shellcode into an executable
sudo apt install build-essential

# libemu is required for outputting a graph using Libemu's sctest
sudo apt install graphviz
sudo apt install libemu2

# gcc-multilib is required to compile x86 executables on x64 systems
apt-get install gcc-multilib

# create symbolic link to YARA library in case it isn't found
ln -s /usr/local/lib/python3.8/dist-packages/usr/lib/libyara.so /usr/lib/libyara.so
```

## Usage

```
=============================================================
                -- Shellcode Assistant v1.0 --
=============================================================

usage: shellcode_assistant.py [-h] -f FILE [-y YARA] [-o OFFSET] [-t] [-s STDOUT] [-se STDERR] [-x] [-c]
                              [-e] [-d] [-a] [-A] [-R] [-ob]

== Tool to assist a user in writing or modifiying shellcode ==

optional arguments:
  -h, --help            show this help message and exit
  -f FILE, --file FILE  Enter path to file.
  -y YARA, --yara YARA  Enter path to YARA rule.
  -o OFFSET, --offset OFFSET
                        Set hexadecimal program offset.
  -t, --test            Run and test compiled executable.
  -s STDOUT, --stdout STDOUT
                        Check STDOUT for a specific string.
  -se STDERR, --stderr STDERR
                        Check STDERR for a specific string.
  -x, --hexdump         Dump hex code.
  -c, --compile         Compile file using C wrapper.
  -e, --emulate         Emulate and analyse shellcode using Libemu.
  -d, --disassemble     Disassemble shellcode.
  -a, --assembler       Start ASM interactive assembler / disassembler mode.
  -A, --all             Run all functions above.
  -R, --autorecompile   Automatic recompiling and testing upon file modification time change.
  -ob, --objdump        Use Linux objdump instead of Python Capstone library.
```

*The Python Capstone library might truncate code in certain cases. Use --objdump if necessary.*

## Examples

```
root@kali:~/OT-Project# python3 shellcode_assistant.py -f testing.bin -R -s "Hello World" -o 0x0 -y rule.yar
root@kali:~/OT-Project# python3 shellcode_assistant.py -f testing.bin -A -s "Hello World"
```

### Demo - Modifying Shikata Ga Nai Encoded 'Hello World' Shellcode

<img title="" src="https://raw.githubusercontent.com/alexander-47u/Shellcode-Assistant/master/demo.gif" width="1000px" height="565" alt="" data-align="center">

**Diagram generated by Libemu**

<img title="" src="https://raw.githubusercontent.com/alexander-47u/Shellcode-Assistant/master/demo.png" width="600px" height="484" alt="" data-align="center">

### ASM Interactive Assembler / Disassembler

```
...
0x40404064    2    31 DB               xor    ebx, ebx
0x40404066    2    CD 80               int    0x8
---------------------------------------
Last Modified: Sat May 16 16:48:48 2020
---------------------------------------
asm > ?
 <x86-instruction> | recompile | test | emulate | yarascan | hexdump | ls | clear | reset | exit
asm > \xC7\xC3\xAC\xCB\x77\x92
mov	ebx, 0x9277cbac
asm > push 0x9277cbac; pop ebx
\x68\xAC\xCB\x77\x92\x5B
asm >

Shellcode emulated:
Graph saved as 'testing.bin.dot' & 'testing.bin.png'
---------------------------------------
asm >
```

### Obtaining Raw Shellcode

#### Method 1 - Compile ASM and dump code section to hex format

```
1. Compile ASM Code
nasm -f elf32 helloworld.asm -o helloworld.o

2. Dump object code to hex code
objdump -d ./helloworld.o|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'

3. Save hex shellcode to raw shellcode file using echo
echo -e "\x31\xc0\xb0\x04\x31\xdb\xb3\x01\x31\xd2\x52\x68\x72\x6c\x64\x0a\x68\x6f\x20\x57\x6f\x68\x48\x65\x6c\x6c\x89\xe1\xb2\x0c\xcd\x80\x31\xc0\xb0\x01\x31\xdb\xcd\x80" > testing.bin
```

#### Method 2 - Use Metasploit's shellcode generator

```
msfvenom -p linux/x86/shell_reverse_tcp LHOST=127.0.0.1 LPORT=443 -f raw -b "\x00" -e x86/shikata_ga_nai -o testing.bin
```

## Authors

Alexander-OS3

Jelle-OS3

## TO-DO

* Add support for Linux x64
* Create script with Windows support
