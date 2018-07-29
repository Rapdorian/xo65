xo65 File Desciption
====================

Why
---

I wrote this because I couldn't find any documentation online for this
file format. I spent several days piecing it together with a hex editor,
cc65 tools, and reading source code. I wouldn't wish that on anyone so I
have put this document together as a reference for myself and others who
might need it.

I am still working on this and many things are missing, anything in here
marked with a ??? I don't currently know.

Type definitions
----------------

    BYTE:   8-bits
    WORD:   16-bits
    DWORD:  32-bits

Header
------

The header is rather simple and consists of three sections. The magic
number and version are constant. However the flags word can indicate if
the file was compiled with debug symbols

    55 7A 6E 61     ; magic number
    11 00           ; version
    00 00           ; flags

Block Table
-----------

The Block Table contains the offset and size of all the rest of the
blocks in the file. Both values are stored as a `DWORD`. cc65 tools
appear to not work with block sizes of 0, so the best way of leaving a
block empty is to have it point at an empty `DWORD`. I have also found
that you can have multiple blocks point to the same data.

    DWORD   ; offset
    DWORD   ; size

These entries are ordered in a constant manner

-   Options
-   Files
-   Segments
-   Imports
-   Exports
-   Debug Symbols
-   Line Info
-   String Pool
-   Assertions
-   Scopes
-   Span?

Segments Block
--------------

The segments block store machine code in a number of relocatable
segments. A segment if made up of fragments, which are machine
instructions that have had there addresses made into expressions instead
of absolutes.

    BYTE        ; number of segments
    SEGMENT[]   ; segments ( see below )

### Segment

    DWORD       ; segment length in bytes
    BYTE        ; name of segment ( index of string in string pool )
    00          ; flags ( no flags defined yet )
    BYTE        ; final code length in bytes
    BYTE        ; alignment
    BYTE        ; address size
    BYTE        ; number of fragments
    FRAG[]      ; fragments

### Fragment

There are three types of fragments: Literal, Expression, and Signed
Expression. The type of a fragment can be determined by masking the
first byte of the fragment with `0x38`. Expressions can also vary in the
size of the value they compute to. The number of bytes an expression
represents can be computed by masking the first byte with `0x07`.

    BYTE        ; type ( mask with 0x38 for type ) and ( mask with 0x07 for byte length )
                ; Literal:      0x00
                ; Expression:       0x08
                ; Signed Expression:    0x10
    DATA[]      ; data ( format varies by type )
    LINEINFO    ; Line info ( see below )

### Literal Format

    BYTE    ; number of bytes in literal
    BYTE[]  ; literal

### Signed Expression Format

I haven't done any testing on signed expressions yet

### Expression Format

An expression can be one of three types: Leaf, Binary, and Unary. Binary
Expressions have two arguments and Unary have one. Arguments to Binary
and Unary expressions can themselves be expressions. Leaf node arguments
vary, however leaf node arguments can't be expressions.

The Type of an expression can be determined by masking it with `0xC0`
and the specific operation can by masking it with `0x3F`

    Leaf Nodes: 0x80

    0x01 LITERAL        ( WORD ) ; word literal
    0x02 SYMBOL         ( BYTE ) ; represents the index of a imported symbol ???
    0x03 SECTION        ( BYTE ) ; represents the index of a section in this file
    0x04 SEGMENT        ( ???  ) ; linker only ???
    0x05 MEMAREA        ( ???  ) ; linker only ???
    0x06 ULABEL         ( ???  ) ; linker only ???

    Binary Nodes: 0x00

    0x01 ADD
    0x02 SUBTRACT
    0x03 MULTIPLY
    0x04 DIVIDE
    0x05 MODULO
    0x06 BITWISE OR
    0x07 BITWISE XOR
    0x08 BITWISE AND
    0x09 LEFT SHIFT
    0x0A RIGHT SHIFT
    0x0B EQUALS
    0x0C NOT EQUAL
    0x0D LESS THAN
    0x0E GREATER THAN
    0x0F LESS THAN OR EQUAL
    0x10 GREATER THAN OR EQUAL
    0x11 LOGICAL AND
    0x12 LOGICAL OR
    0x13 LOGICAL XOR
    0x14 MAX
    0x15 MIN

    Unary Nodes: 0x40

    0x01 NEGATIVE
    0x02 BITWISE NOT
    0x03 SWAP           ; ???
    0x04 LOGICAL NOT
    0x05 BANK           ; ???
    ; 
    0x08 BYTE 0
    0x09 BYTE 1
    0x0A BYTE 2
    0x0B BYTE 3
    0x0C WORD 0
    0x0D WORD 1
    0x0E FAR ADDRESS    ; ???
    0x0F DWORD          ; ???

### Line Info

Line info just mentions which source line something is on

    BYTE    ; number of bytes in info block
    BYTE[]  ; index into line info block ???

String Pool Block
-----------------

The string pool block is a list of all the string used in the file.

    WORD        ; number of strings ???
    STRING[]    ; strings ( see below )

### Strings

    BYTE        ; string length
    BYTE[]      ; chars
