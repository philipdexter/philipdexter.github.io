---
layout: post
title: Example of Type-Based Optimizations in Factor
---

Factor is dynamically typed, yet with words such as
[declare](http://docs.factorcode.org/content/word-declare,kernel.private.html)
and
[TYPED:](http://docs.factorcode.org/content/word-TYPED__colon__,typed.html)
we can guide the compiler to generate more efficient code.

Consider the following two words and their definitions:

```factor
: add-untyped ( x y -- z ) + ;
TYPED: add-typed ( x: fixnum y: fixnum -- z: fixnum ) + ;
```

We will use the
[disassemble](http://docs.factorcode.org/content/word-disassemble%2Ctools.disassembler.html)
word for viewing assembly.
Let's now compare the difference between `add-untyped`
and `add-typed`.
First, `add-untyped`:
```asm
00007f134c061820: 8905da27f4fe    mov [rip-0x10bd826], eax
00007f134c061826: 8905d427f4fe    mov [rip-0x10bd82c], eax
00007f134c06182c: 488d1d05000000  lea rbx, [rip+0x5]
00007f134c061833: e9e8f115ff      jmp 0x7f134b1c0a20 (+)
```
Very simple, this jumps to the generic word `+`
which will dynamically dispatch to the appropriate `+` definition for `fixnum`.
And now `add-typed`:
```asm
00007f134c041c90: 89056a23f6fe    mov [rip-0x109dc96], eax
00007f134c041c96: 53              push rbx
00007f134c041c97: 498b06          mov rax, [r14]
00007f134c041c9a: 4983ee08        sub r14, 0x8
00007f134c041c9e: 4983c708        add r15, 0x8
00007f134c041ca2: 498907          mov [r15], rax
00007f134c041ca5: e8d65dfefe      call 0x7f134b027a80 ([ \ integer>fixnum-strict ~array~ 0 ~array~ inline-cache-miss ])
00007f134c041caa: 498b07          mov rax, [r15]
00007f134c041cad: 4983c608        add r14, 0x8
00007f134c041cb1: 4983ef08        sub r15, 0x8
00007f134c041cb5: 498906          mov [r14], rax
00007f134c041cb8: e8c35dfefe      call 0x7f134b027a80 ([ \ integer>fixnum-strict ~array~ 0 ~array~ inline-cache-miss ])
00007f134c041cbd: 89053d23f6fe    mov [rip-0x109dcc3], eax
00007f134c041cc3: 5b              pop rbx
00007f134c041cc4: 488d1d05000000  lea rbx, [rip+0x5]
00007f134c041ccb: e950ffffff      jmp 0x7f134c041c20 (( typed add-typed ))
```
This first tries to coerce the two inputs to `fixnum` and then jumps to an
address marked as `(( typed add-typed ))`. The assembly for `(( typed add-typed ))` is
```asm
00007f134c041c20: 8905da23f6fe    mov [rip-0x109dc26], eax
00007f134c041c26: 53              push rbx
00007f134c041c27: 498b06          mov rax, [r14]
00007f134c041c2a: 498b5ef8        mov rbx, [r14-0x8]
00007f134c041c2e: 4801c3          add rbx, rax
00007f134c041c31: 0f801a000000    jo 0x7f134c041c51 (( typed add-typed ) + 0x31)
00007f134c041c37: 4983ee08        sub r14, 0x8
00007f134c041c3b: 49891e          mov [r14], rbx
00007f134c041c3e: 8905bc23f6fe    mov [rip-0x109dc44], eax
00007f134c041c44: 5b              pop rbx
00007f134c041c45: 488d1d05000000  lea rbx, [rip+0x5]
00007f134c041c4c: e96f5dfefe      jmp 0x7f134b0279c0 ([ \ integer>fixnum-strict ~array~ 0 ~array~...)
00007f134c041c51: e80a1a45ff      call 0x7f134b493660 (fixnum+overflow)
00007f134c041c56: 8905a423f6fe    mov [rip-0x109dc5c], eax
00007f134c041c5c: 5b              pop rbx
00007f134c041c5d: 488d1d05000000  lea rbx, [rip+0x5]
00007f134c041c64: e9575dfefe      jmp 0x7f134b0279c0 ([ \ integer>fixnum-strict ~array~ 0 ~array~...)
```
This will do an `add` instruction directly and then check for overflow.
No dynamic dispatch involved.

Let's test these out with some input values.
We'll look at the assembly for `[1 2 add-untyped]` and then `[1 2 add-typed]`.
First, using `add-untyped`:
```asm
00007f134c148370: 89058abce5fe      mov [rip-0x11a4376], eax
00007f134c148376: 4983c610          add r14, 0x10
00007f134c14837a: 49c70620000000    mov qword [r14], 0x20
00007f134c148381: 49c746f810000000  mov qword [r14-0x8], 0x10
00007f134c148389: 890571bce5fe      mov [rip-0x11a438f], eax
00007f134c14838f: 488d1d05000000    lea rbx, [rip+0x5]
00007f134c148396: e98594f1ff        jmp 0x7f134c061820 (add-untyped)
00007f134c14839b: 0000              add [rax], al
00007f134c14839d: 0000              add [rax], al
00007f134c14839f: 00                invalid
```
As expected, this jumps to `add-untyped`.
And now using `add-typed`:
```asm
00007f134c146850: 8905aad7e5fe      mov [rip-0x11a2856], eax
00007f134c146856: 4983c610          add r14, 0x10
00007f134c14685a: 49c70620000000    mov qword [r14], 0x20
00007f134c146861: 49c746f810000000  mov qword [r14-0x8], 0x10
00007f134c146869: 890591d7e5fe      mov [rip-0x11a286f], eax
00007f134c14686f: 488d1d05000000    lea rbx, [rip+0x5]
00007f134c146876: e9a5b3efff        jmp 0x7f134c041c20 (( typed add-typed ))
00007f134c14687b: 0000              add [rax], al
00007f134c14687d: 0000              add [rax], al
00007f134c14687f: 00                invalid
```
This directly jumps to `(( typed add-typed ))` which will directly do the `add` instruction.
The compiler safely skipped the type coercion as it knows the inputs are both `fixnum`.
