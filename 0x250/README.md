<div style="text-align: center;">
    <img src="../img/x86_64_intel.png" width="240">
</div>

# Désassemblage
L'option **-M intel** permet d'avoir un désassemblage selon la syntaxe Intel plutôt qu'AT&T.
```bash
$ objdump -M intel -D firstprog | grep -A20 main.:
0000000000001139 <main>:
    1139:       55                      push   rbp
    113a:       48 89 e5                mov    rbp,rsp
    113d:       48 83 ec 10             sub    rsp,0x10
    1141:       c7 45 fc 00 00 00 00    mov    DWORD PTR [rbp-0x4],0x0
    1148:       eb 13                   jmp    115d <main+0x24>
    114a:       48 8d 05 b3 0e 00 00    lea    rax,[rip+0xeb3]        # 2004 <_IO_stdin_used+0x4>
    1151:       48 89 c7                mov    rdi,rax
    1154:       e8 d7 fe ff ff          call   1030 <puts@plt>
    1159:       83 45 fc 01             add    DWORD PTR [rbp-0x4],0x1
    115d:       83 7d fc 09             cmp    DWORD PTR [rbp-0x4],0x9
    1161:       7e e7                   jle    114a <main+0x11>
    1163:       b8 00 00 00 00          mov    eax,0x0
    1168:       c9                      leave
    1169:       c3                      ret
```

# Registres 64 bits
- registres généraux : RAX, RCX, RDX et RBX (**A**cummulator,**C**ounter, **D**ata et **B**ase)
- registres pointeurs et index : RSP, RBP, RSI et RDI (**S**tack **P**ointer, **B**ase **P**ointer, **S**ource **I**ndex et **D**estination **I**ndex)
- pointeur d'instruction : RIP (**I**nstruction **P**ointer)
- registre EFLAGS : constitués de drapeaux (flag), bits indicateurs d'état
``` bash
$ gdb -q firstprog
Reading symbols from firstprog...
(gdb) break main
Breakpoint 1 at 0x1141: file /home/electro/workspace/hacking-techniques-x86-64/0x250/firstprog.c, line 6.
(gdb) run
Starting program: /home/electro/workspace/hacking-techniques-x86-64/0x250/firstprog 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, main () at /home/electro/workspace/hacking-techniques-x86-64/0x250/firstprog.c:6
warning: Source file is more recent than executable.
6           for (i = 0; i < 10; i++)
(gdb) i r
rax            0x555555555139      93824992235833
rbx            0x7fffffffd928      140737488345384
rcx            0x555555557dd8      93824992247256
rdx            0x7fffffffd938      140737488345400
rsi            0x7fffffffd928      140737488345384
rdi            0x1                 1
rbp            0x7fffffffd810      0x7fffffffd810
rsp            0x7fffffffd800      0x7fffffffd800
r8             0x0                 0
r9             0x7ffff7fcf680      140737353938560
r10            0x7ffff7fcb878      140737353922680
r11            0x7ffff7fe1940      140737354012992
r12            0x0                 0
r13            0x7fffffffd938      140737488345400
r14            0x555555557dd8      93824992247256
r15            0x7ffff7ffd020      140737354125344
rip            0x555555555141      0x555555555141 <main+8>
eflags         0x206               [ PF IF ]
```
# Désassemblage avec gdb
Le code source est affiché puis vient la version désassemblée de la fonction main(). Un point d'arrêt est fixé au début du main() et le programme est démarré. La valeur de RIP est alors affichée, il s'agit de l'adresse <main+8>. Le bloc qui précède appelé prologue de fonction permet de réserver de la mémoire pour les variables locales (pile -> stack).
Si l'on veut voir le prologue il faut taper :
```bash
(gdb) break *main
```

```bash
$ gdb -q firstprog
Reading symbols from firstprog...
(gdb) list
1       #include <stdio.h>
2
3       int main()
4       {
5           int i;
6           for (i = 0; i < 10; i++)
7           {
8               printf("Hello, world!\n");
9           }
10          return 0;
(gdb) disass main
Dump of assembler code for function main:
   0x0000000000001139 <+0>:     push   rbp
   0x000000000000113a <+1>:     mov    rbp,rsp
   0x000000000000113d <+4>:     sub    rsp,0x10
   0x0000000000001141 <+8>:     mov    DWORD PTR [rbp-0x4],0x0
   0x0000000000001148 <+15>:    jmp    0x115d <main+36>
   0x000000000000114a <+17>:    lea    rax,[rip+0xeb3]        # 0x2004
   0x0000000000001151 <+24>:    mov    rdi,rax
   0x0000000000001154 <+27>:    call   0x1030 <puts@plt>
   0x0000000000001159 <+32>:    add    DWORD PTR [rbp-0x4],0x1
   0x000000000000115d <+36>:    cmp    DWORD PTR [rbp-0x4],0x9
   0x0000000000001161 <+40>:    jle    0x114a <main+17>
   0x0000000000001163 <+42>:    mov    eax,0x0
   0x0000000000001168 <+47>:    leave
   0x0000000000001169 <+48>:    ret
End of assembler dump.
(gdb) break main
Breakpoint 1 at 0x1141: file /home/electro/workspace/hacking-techniques-x86-64/0x250/firstprog.c, line 6.
(gdb) run
Starting program: /home/electro/workspace/hacking-techniques-x86-64/0x250/firstprog 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, main () at /home/electro/workspace/hacking-techniques-x86-64/0x250/firstprog.c:6
6           for (i = 0; i < 10; i++)
(gdb) i r rip
rip            0x555555555141      0x555555555141 <main+8>
```
# Exemples d'utilisation de la commande x dans gdb :
- Format de la commande : x/[<nombre d'unités>]<Format d'affichage>[<Unité>] <Adresse mémoire>
- Format d'affichage :
```
o : octal
x : o :hexadécimal
d : décimal
c : caractère (pas d'unité)
s : chaîne de caractères (pas d'unité)
i : instruction (pas d'unité)
```
- Unité :
```
b : octet (byte)
h : demi-mot (2 octets)
w : mot (4 octets)
g : double mot (8 octets)
```
- Exemple (l'unité est w par défaut)
```bash
(gdb) x/2x $rip
0x555555555141 <main+8>:        0x00fc45c7      0xeb000000
(gdb) x/8xb $rip
0x555555555141 <main+8>:        0xc7    0x45    0xfc    0x00    0x00    0x00    0x00    0xeb
(gdb)
```
> On remarque dans l'exemple précédent que l'orientation des octets est de type little-endian i.e. octet de poids faible en premier.

# Convention d'appels
La convention d'appel pour une architecture Linux x86_64 est différente de l'architecture Linux x86 :
| Élément | Registre |
|----------|----------|
| Argument 1 | `RDI` |
| Argument 2 | `RSI` |
| Argument 3 | `RDX` |
| Argument 4 | `RCX` |
| Argument 5 | `R8` |
| Argument 6 | `R9` |
| Arguments suivants | Pile (`RSP`) |
| Valeur de retour | `RAX` |
| Retour 128 bits | `RDX:RAX` |
```bash
(gdb)
   0x55555555514a <main+17>:    lea    rax,[rip+0xeb3]        # 0x555555556004
   0x555555555151 <main+24>:    mov    rdi,rax
   0x555555555154 <main+27>:    call   0x555555555030 <puts@plt>
(gdb) nexti
0x0000555555555151      8               printf("Hello, world!\n");
(gdb) print $rip+0xeb3
$1 = (void (*)()) 0x555555556004
(gdb) x/s $1
0x555555556004: "Hello, world!"
```
> Le registre RDI pointe vers "Hello, world!" avant l'appel de puts@plt, cela est conforme à l'abi Linux x86_64.
> On peut également vérifier que l'adresse de la chaîne de caractères est bien dans la section .rodata :
```bash
(gdb) info files
Symbols from "/home/electro/workspace/hacking-techniques-x86-64/0x250/firstprog".
Native process:
        Using the running image of child Thread 0x7ffff7dd2740 (LWP 81554).
        While running this, GDB does not access memory from...
Local exec file:
        `/home/electro/workspace/hacking-techniques-x86-64/0x250/firstprog', file type elf64-x86-64.
        Entry point: 0x555555555050
        0x0000555555554318 - 0x0000555555554334 is .interp
        0x0000555555554338 - 0x0000555555554358 is .note.gnu.property
        0x0000555555554358 - 0x000055555555437c is .note.gnu.build-id
        0x000055555555437c - 0x000055555555439c is .note.ABI-tag
        0x00005555555543a0 - 0x00005555555543c4 is .gnu.hash
        0x00005555555543c8 - 0x0000555555554470 is .dynsym
        0x0000555555554470 - 0x00005555555544fd is .dynstr
        0x00005555555544fe - 0x000055555555450c is .gnu.version
        0x0000555555554510 - 0x0000555555554540 is .gnu.version_r
        0x0000555555554540 - 0x0000555555554600 is .rela.dyn
        0x0000555555554600 - 0x0000555555554618 is .rela.plt
--Type <RET> for more, q to quit, c to continue without paging--
        0x0000555555555000 - 0x0000555555555017 is .init
        0x0000555555555020 - 0x0000555555555040 is .plt
        0x0000555555555040 - 0x0000555555555048 is .plt.got
        0x0000555555555050 - 0x000055555555516a is .text
        0x000055555555516c - 0x0000555555555175 is .fini
        0x0000555555556000 - 0x0000555555556012 is .rodata
        0x0000555555556014 - 0x0000555555556040 is .eh_frame_hdr
        0x0000555555556040 - 0x00005555555560ec is .eh_frame
        0x0000555555557dd0 - 0x0000555555557dd8 is .init_array
        0x0000555555557dd8 - 0x0000555555557de0 is .fini_array
        0x0000555555557de0 - 0x0000555555557fc0 is .dynamic
        0x0000555555557fc0 - 0x0000555555557fe8 is .got
        0x0000555555557fe8 - 0x0000555555558008 is .got.plt
        0x0000555555558008 - 0x0000555555558018 is .data
        0x0000555555558018 - 0x0000555555558020 is .bss
        0x00007ffff7fcb238 - 0x00007ffff7fcb25c is .note.gnu.build-id in /lib64/ld-linux-x86-64.so.2
        0x00007ffff7fcb260 - 0x00007ffff7fcb39c is .hash in /lib64/ld-linux-x86-64.so.2
        0x00007ffff7fcb3a0 - 0x00007ffff7fcb500 is .gnu.hash in /lib64/ld-linux-x86-64.so.2
```
