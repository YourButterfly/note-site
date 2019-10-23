# Program State

## State Plugins

### history plugin

```text
    It is actually a linked list of several history nodes, each one representing a single round of execution---you can traverse this list with state.history.parent.parent etc.

    In general, these values are stored as history.recent_NAME and the iterator over them is just history.NAME. For example, for addr in state.history.bbl_addrs: print hex(addr) will print out a basic block address trace for the binary, while state.history.recent_bbl_addrs is the list of basic blocks executed in the most recent step(最近的一步不应该是一个吗), state.history.parent.recent_bbl_addrs is the list of basic blocks executed in the previous step, etc.

    If you ever need to quickly obtain a flat list of these values, you can access .hardcopy, e.g. state.history.bbl_addrs.hardcopy. Keep in mind though, index-based accessing is implemented on the interators.
```

```python
    for addr in state.history.bbl_addrs:
        print(hex(addr))
    print("--------------------------------\n")
"""
------------------------------------
0x40137b
0x4013e0
...
0x401b08
0x401b36
------------------------------------
0x40137b
0x4013e0
...
0x401b08
0x401b36
------------------------------------
"""

    for addr in state.history.recent_bbl_addrs:
        print(hex(addr))
    print("--------------------------------\n")
"""

------------------------------------
0x401b36
------------------------------------
0x401b36
------------------------------------
"""

    print(state.history.bbl_addrs.hardcopy)
"""
...
[4199291, 4199392, 4199405, 4199426, 4199467, 4199495, 4199507, 4199560, 4199607, 4201284, 4199619, 4199660, 4199676, 4199717, 4199733, 4199774, 4199790, 4199831, 4199847, 4199888, 4199904,4199945, 4199961, 4200002, 4200018, 4200059, 4200075, 4200116, 4200132, 4200173, 4200189, 4200230, 4200246, 4200287, 4200303, 4200344, 4200360, 4200401, 4200417, 4200458, 4200474, 4200515, 4200531, 4200572, 4200588, 4200629, 4200645, 4200686, 4200702, 4200743, 4200759, 4200800, 4200816, 4200857, 4200945, 4200986, 4201002, 4201043, 4201059, 4201100, 4201116, 4201157, 4201170, 4201211, 4201224, 4201270]
"""
```

```text
    history.descriptions is a listing of string descriptions of each of the rounds of execution performed on the state.
    history.bbl_addrs 这个state执行过的基本块的地址的有序列表.
    There may be more than one per round of execution, and not all addresses may correspond to binary code - some may be addresses at which SimProcedures are hooked.
    history.jumpkinds 在state的history中每个控制流转变处理的有序列表, as VEX enum strings.
    history.guards is a listing of the conditions guarding each of the branches that the state has encountered.
    history.events is a semantic listing of "interesting events" which happened during execution, such as the presence of a symbolic jump condition, the program popping up a message box, or execution terminating with an exit code.
    history.actions 通常为空, 但如果你增加了 angr.options.refs 选项给 state, 它的内容是 log of all the memory, register, and temporary value accesses performed by the program.
```

```python
    for i in state.history.descriptions:
        print(i)
"""
---------------------------
<IRSB from 0x40137b: 1 sat 1 unsat>
...
<IRSB from 0x401afb: 2 sat>
<IRSB from 0x401b08: 1 sat 1 unsat>
<IRSB from 0x401b36: 1 sat 1 unsat>
---------------------------
"""
for i in state.history.actions:
    print(i)
"""
---------------------------
很多很长
...
<SimActionData 0x401b36:8 tmp/write>
<SimActionExit 0x401b36:10 default>
<SimActionConstraint 0x401b36:10 <SAO <Bool True>>>
<SimActionConstraint 0x401b36:10 <SAO <Bool False>>>
---------------------------
"""
for i in state.history.jumpkinds:
    print(i)
print("---------------------------")
print(state.history.jumpkind)
"""
...
Ijk_Boring
Ijk_FakeRet
Ijk_Call
---------------------------
Ijk_Call
"""
```
