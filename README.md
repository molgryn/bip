# Bip

Bip is a project which aimed to simplify the usage of python for interacting
with IDA. Its main goals are to facilitate the usage of python in the
interactive console of IDA and for writting plugins. In a more general way
the goal is to automate the recurent task done through the python API.
Bip is also developped for providing a more oriented object, a "python-like"
API and a *real* documentation.

This code is not complete and a lot of features are still missing. Development
is prioritize on what people ask for and what the developers use, so do not
hesitate to make PR and Issue (including feature request :).

The documentation is available in the RST format (and can be compile using
sphinx) in the `doc/` directory.

## Installation

TODO (this is a classic IDA plugin install, no setup.py yet)

## Overview

This overview has for goal to show how the most
usual operations can be done, it is far from being complete. 

### Base

The module `bip.base` contains most of the *basic* features for interfacing
with IDA. This includes: manipulation of instruction, functions, basic blocks,
operands, data, xrefs, structures, types, ...

#### Instructions / Operands

``` python
>>> from bip.base import *
>>> i = Instr() # Instr is the base class for representing an instruction
>>> i # by default the address on the screen is taken
Instr: 0x1800D324B (mov     rcx, r13)
>>> i2 = Instr(0x01800D3242) # pass the address in argument
>>>> i2
Instr: 0x1800D3242 (mov     r8d, 8)
>>> i2.next # access next instruction, preivous with i2.prev
Instr: 0x1800D3248 (mov     rdx, r14)
>>> l = [i3 for i3 in Instr.iter_all()] # l contain the list of all Instruction of the database, iter_all produce a generator object
>>> i.ea # access the address
6443315787
>>> i.mnem # mnemonic representation
mov
>>> i.ops # access to the operands
[<bip.base.operand.Operand object at 0x0000022B0291DA90>, <bip.base.operand.Operand object at 0x0000022B0291DA58>]
>>> i.ops[0].str # string representation of an operand
rcx
>>> i.bytes # bytes in the instruction
[73L, 139L, 205L]
>>> i.size # number of bytes of this instruction
3
>>> i.comment = "hello" # set a comment, rcomment for the repeatable comments
>>> i
Instr: 0x1800D324B (mov     rcx, r13; hello)
>>> i.comment # get a comment
hello
>>> i.func # access to the function
Func: RtlQueryProcessLockInformation (0x1800D2FF0)
>>> i.block # access to basic block
BipBlock: 0x1800D3242 (from Func: RtlQueryProcessLockInformation (0x1800D2FF0))
```

#### Function / Basic block

```python
>>> from bip.base import *
>>> f = BipFunction() # Get the function, screen address used if not provided
>>> f
Func: RtlQueryProcessLockInformation (0x1800D2FF0)
>>> f2 = BipFunction(0x0018010E975) # provide an address, not necessary the first one
>>> f2
Func: sub_18010E968 (0x18010E968)
>>> f == f2 # compare two functions
False
>>> f == BipFunction(0x001800D3021)
True
>>> hex(f.ea) # start address
0x1800d2ff0L
>>> hex(f.end) # end address
0x1800d3284L
>>> f.name # get and set the name
RtlQueryProcessLockInformation
>>> f.name = "test"
>>> f.name
test
>>> f.size # number of bytes in the function
660
>>> f.bytes # bytes of the function
[72L, ..., 255L]
>>> f.callees # list of function called by this function
[<bip.base.func.BipFunction object at 0x0000022B0291DD30>, ..., <bip.base.func.BipFunction object at 0x0000022B045487F0>]
>>> f.callers # list of function which call this function
[<bip.base.func.BipFunction object at 0x0000022B04544048>]
>>> f.instr # list of instructions in the function
[<bip.base.instr.Instr object at 0x0000022B0291DB00>, ..., <bip.base.instr.Instr object at 0x0000022B0454D080>]
>>> f.comment = "welcome to bip" # comment of the function, rcomment for repeatables one 
>>> f.comment
welcome to bip
>>> f.does_return # does this function return ?
True
>>> BipFunction.iter_all() # allow to iter on all functions define in the database
<generator object iter_all at 0x0000022B029231F8>
>>> f.nb_blocks # number of basic block
33
>>> f.blocks # list of blocks
[<bip.base.block.BipBlock object at 0x0000022B04544D68>, ..., <bip.base.block.BipBlock object at 0x0000022B04552240>]
>>> f.blocks[5] # access the basic block 5, could be done with BipBlock(addr)
BipBlock: 0x1800D306E (from Func: test (0x1800D2FF0))
>>> f.blocks[5].func # link back to the function
Func: test (0x1800D2FF0)
>>> f.blocks[5].instr # list of instruction in the block
[<bip.base.instr.Instr object at 0x0000022B04544710>, ..., <bip.base.instr.Instr object at 0x0000022B0291DB00>]
>>> f.blocks[5].pred # predecessor blocks, blocks where control flow lead to this one
[<bip.base.block.BipBlock object at 0x0000022B04544D68>]
>>> f.blocks[5].succ # successor blocks
[<bip.base.block.BipBlock object at 0x0000022B04544710>, <bip.base.block.BipBlock object at 0x0000022B04544438>]
>>> f.blocks[5].is_ret # is this block containing a return
False
```

#### Data

``` python
>>> from bip.base import *
>>> d = BipData(0x000180110068) # .rdata:0000000180110068 bip_ex          dq offset unk_180110DE0
BipData at 0x180110068 = 0x180110DE0
>>> d.name # Name of the symbol if any
bip_ex
>>> d.is_word # is it a word
False
>>> d.is_qword # is it a qword
True
>>> hex(d.value) # value at that address, this take into account the basic type (byte, word, dword, qword) defined in IDA
0x180110de0L
>>> hex(d.ea) # address
0x180110068L
>>> d.comment = "exemple" # comment as before
>>> d.comment
exemple
>>> d.value = 0xAABBCCDD # change the value 
>>> hex(d.value)
0xaabbccddL
>>> d.bytes # get the bytes, as before
[221L, 204L, 187L, 170L, 0L, 0L, 0L, 0L]
>>> hex(d.original_value) # get the original value before modification
0x180110de0L
>>> d.bytes = [0x11, 0x22, 0x33, 0x44, 0, 0, 0, 0] # patch the bytes
>>> hex(d.value) # get the value
0x44332211L
>>> BipData.iter_heads() # iter on "heads" of the IDB, heads are defined data in the IDB
<generator object iter_heads at 0x0000022B02923240>
>>> hex(BipData.get_dword(0x0180110078)) # staticmethod for reading value at an address
0x60004L
>>> BipData.set_byte(0x0180110078, 0xAA) # static method for modifying a value at an address
>>> hex(BipData.get_qword(0x0180110078))
0x600aaL
```

#### Element

In Bip most basic object inherit from the same classes: `BipBaseElt` which is
the most basic one, `BipRefElt` which include all the objects which can have
xrefs (including structures and structure members, see bellow), `BipElt`
which represent all elements which have an address in the IDA DataBase (idb),
including `BipData` and `Instr` (it is this class which implement the
properties `comment`,  `name`, `bytes`, ...).

It is possible to use the functions `GetElt` and `GetEltByName` for directly
recuperating the good basic element from an address or a name representing
a location in the binary.

``` python
>>> from bip.base import *
>>> GetElt() # get the element at current address, in that case return a BipData object
BipData at 0x180110068 = 0x44332211
>>> GetElt(0x00180110078) # get the element at the address 0x00180110078
BipData at 0x180110078 = 0xAA
>>> GetElt(0x1800D2FF0) # in that case it return an Instr object because this is code
Instr: 0x1800D2FF0 (mov     rax, rsp)
>>> GetEltByName("bip_ex") # Get using a name and not an address
BipData at 0x180110068 = 0x44332211
>>> isinstance(GetElt(0x1800D2FF0), Instr) # test if that element is an instruction ?
True
>>> isinstance(GetElt(0x1800D2FF0), BipData) # or data ?
False
```

#### Xref

All elements which inherit from `BipRefElt` (`Instr`, `BipData`, `BipStruct`,
...) and some other (in particular `BipFunction`) possess methods which allow
to access xrefs. They are represented by the `BipXref` object which have a
`src` (origin of the xref) and a `dst` (destination of the xref).

``` python
>>> i = Instr(0x01800D3063)
>>> i # exemple with instruction but works the same with BipData
Instr: 0x1800D3063 (cmp     r15, [rsp+98h+var_58])
>>> i.xTo # List of xref which point on this instruction
[<bip.base.xref.BipXref object at 0x0000022B04544438>, <bip.base.xref.BipXref object at 0x0000022B045447F0>]
>>> i.xTo[0].src # previous instruction
Instr: 0x1800D305E (mov     [rsp+98h+var_78], rsi)
>>> i.xTo[0].is_ordinaryflow # is this an ordinary flow between to instruction (not jmp or call)
True
>>> i.xTo[1].src # jmp to instruction i at 0x1800D3063
Instr: 0x1800D3222 (jmp     loc_1800D3063)
>>> i.xTo[1].is_jmp # is this xref because of a jmp ?
True
>>> i.xEaTo # bypass the xref objects and get the address directly
[6443315294L, 6443315746L]
>>> i.xEltTo # bypass the xref objects and get the elements directly, will list BipData if any
[<bip.base.instr.Instr object at 0x0000022B045447F0>, <bip.base.instr.Instr object at 0x0000022B04544978>]
>>> i.xCodeTo # bypass the xref objects and get the instr directly, if a BipData was pointed at this address it will not be listed
[<bip.base.instr.Instr object at 0x0000022B04544438>, <bip.base.instr.Instr object at 0x0000022B0291DD30>]
>>> i.xFrom # same but for comming from this instruction
[<bip.base.xref.BipXref object at 0x0000022B04544D68>]
>>> i.xFrom[0]
<bip.base.xref.BipXref object at 0x0000022B04544438>
>>> i.xFrom[0].dst # next instruction
Instr: 0x1800D3068 (jz      loc_1800D3227)
>>> i.xFrom[0].src # current instruction
Instr: 0x1800D3063 (cmp     r15, [rsp+98h+var_58])
>>> hex(i.xFrom[0].dst_ea) # address of the next instruction
0x1800D3068L
>>> i.xFrom[0].is_codepath # this is a normal code path (include jmp and call)
True
>>> i.xFrom[0].is_call # is this because of a call ?
False
>>> f = BipFunction()
>>> f
Func: RtlQueryProcessLockInformation (0x1800D2FF0)
>>> f.xTo # works also for function, but only with To, not with the From
[<bip.base.xref.BipXref object at 0x000001D95529EB00>, <bip.base.xref.BipXref object at 0x000001D95529EB70>, <bip.base.xref.BipXref object at 0x000001D95529EBE0>, <bip.base.xref.BipXref object at 0x000001D95529EC88>]
>>> f.xEltTo # here we have 3 data reference to this function
[<bip.base.instr.Instr object at 0x000001D95529EE48>, <bip.base.data.BipData object at 0x000001D95529EEF0>, <bip.base.data.BipData object at 0x000001D95529EF28>, <bip.base.data.BipData object at 0x000001D95529EF60>]
>>> f.xCodeTo # but only one instruction
[<bip.base.instr.Instr object at 0x000001D95529EC88>]
```




