# DDG

    数据依赖图分析

## 标题没想好

    ddg.graph 的节点是CodeLocation object

```python
"""
Constructor.

:param int block_addr:      块地址
:param int stmt_idx:        Statement ID. None for SimProcedures
:param class sim_procedure: 对应的 SimProcedure class.
:param int ins_addr:        指令地址. Optional.
:param kwargs:              其他参数, will be stored, but not used in __eq__ or __hash__.
"""

<0x40137b id=0x40137b[3]>
<$ins_addr id=$block_addr[$stmt_idx]>
```

## class DDG(Analysis)

    This is a fast data dependence graph directly generated from our CFG analysis result. The only reason for its
    existence is the speed. There is zero guarantee for being sound or accurate. You are supposed to use it only when
    you want to track the simplest data dependence, and you do not care about soundness or accuracy.

    For a better data dependence graph, please consider performing a better static analysis first (like Value-set
    Analysis), and then construct a dependence graph on top of the analysis result (for example, the VFG in angr).

    Also note that since we are using states from CFG, any improvement in analysis performed on CFG (like a points-to
    analysis) will directly benefit the DDG.

### 类DDG\_\_init\_\_

    __init__(self, cfg, start=None, call_depth=None, block_addrs=None)

相关参数

    cfg:         Control flow graph. Please make sure each node has an associated `state` with it. You may
                            want to generate your CFG with `keep_state=True`.(cfg with state)
    start:       An address, Specifies where we start the generation of this data dependence graph.(分析ddg的起始地址)
    call_depth:  None or integers. A non-negative integer specifies how deep we would like to track in the
                            call tree. None disables call_depth limit.(调用深度限制)
    iterable or None block_addrs: A collection of block addresses that the DDG analysis should be performed
                                             on.(是什么再说)

重要变量

``` python
    # 输入的参数相关
    self._cfg = cfg
    self._start = self.project.entry if start is None else start
    self._call_depth = call_depth
    self._block_addrs = block_addrs

    # 结果存储
    self._stmt_graph = networkx.DiGraph()
    self._data_graph = networkx.DiGraph()
    self._simplified_data_graph = None

    self._ast_graph = networkx.DiGraph()  # A mapping of ProgramVariable to ASTs

    self._symbolic_mem_ops = set()

    # 每个函数的数据依赖图
    self._function_data_dependencies = None

    self.view = DDGView(self._cfg, self, simplified=False)
    self.simple_view = DDGView(self._cfg, self, simplified=True)

    # 其他

```

Begin construction!

    self._construct()

### 构造\_construct(self)

        """
        Construct the data dependence graph.

        We track the following types of dependence:
        - (Intra-IRSB) temporary variable dependencies
        - 寄存器依赖
        - 内存依赖, 尽管功能受限
  
        追踪一下几种内存依赖:
        - (Intra-functional) Stack read/write.
            Trace changes of stack pointers inside a function, and the dereferences of stack pointers.
        - (Inter-functional) Stack read/write.
        - (Global) Static memory positions.
            Keep a map of all accessible memory positions to their source statements per function. After that, we
            traverse the CFG and link each pair of reads/writes together in the order of control-flow.

        We do not track the following types of memory access
        - Symbolic memory access
            Well, they cannot be tracked under fastpath mode (which is the mode we are generating the CTF) anyways.
        """

一些初始化操作

```python
        # 初始化worklist,这一步将要分析的CFG node 加入worklist_set,将DDGJob 加入worklist
        if self._start is None:
            # initial nodes are those nodes in CFG that has no in-degrees
            for n in self._cfg.graph.nodes():
                if self._cfg.graph.in_degree(n) == 0:
                    # Put it into the worklist
                    job = DDGJob(n, 0)
                    self._worklist_append(job, worklist, worklist_set)
        else:
            for n in self._cfg.get_all_nodes(self._start):
                job = DDGJob(n, 0)
                self._worklist_append(job, worklist, worklist_set)
                """
                增加一个CFGNode 和它的后继Node到work-list,这里同样要满足call-depth的限制

                :param node_wrapper:    The NodeWrapper instance to insert.DDGJob
                :param worklist:        The work-list, which is a list.
                :param worklist_set:    A set of all CFGNodes that are inside the work-list, 为了加速查找.
                                        It will be updated as well.
                :returns:               A set of newly-inserted CFGNodes (not NodeWrapper instances).
                # #
                worklist.append(node_wrapper)　DDGJob的实例
                worklist_set.add(node_wrapper.cfg_node)　添加的是cfg　node

                """
```

开始循环处理
分析的对象是每个CFGJob，CFGJob是node和call deepth的封装，操作的是node的final state，call deepth用作限制分析的深度

```python
live_defs_per_node = {}

while worklist:
    # pop 一个节点
    ddg_job = worklist[0]
    l.debug("Processing %s.", ddg_job)
    node, call_depth = ddg_job.cfg_node, ddg_job.call_depth
    worklist = worklist[ 1 : ]
    worklist_set.remove(node)

    # 抓取所有的 final states. 通常超过一个 (对每一个 successor　都有一个state), 并处理它们
    # process all of them
    final_states = node.final_states

    #为每个节点创建livedefinition对象实例，初步判定表示生命周期的数据定义
    if node in live_defs_per_node:
        live_defs = live_defs_per_node[node]
    else:
        live_defs = LiveDefinitions()
        live_defs_per_node[node] = live_defs

    successing_nodes = list(self._cfg.graph.successors(node))

    # 尝试把每个final state分配给对应的后继node，反之亦然
    match_suc = defaultdict(bool)
    match_state = defaultdict(set)

    for suc in successing_nodes:
        matched = False
        for state in final_states:
            try:
                if state.solver.eval(state.ip) == suc.addr:　# 判断这个state的跳转地址是否是后继node的地址
                    match_suc[suc.addr] = True
                    match_state[state].add(suc)
                    matched = True
            except (SimUnsatError, SimSolverModeError, ZeroDivisionError):
                # ignore
                matched = matched
        if not matched:
            break

    # 查看是否所有的final state都找到了它的后继node,以及后继node都有final state
    matches = len(match_suc) == len(successing_nodes) and len(match_state) == len(final_states)

    for state in final_states:
        if not matches and state.history.jumpkind == 'Ijk_FakeRet' and len(final_states) > 1:
            # Skip fakerets if there are other control flow transitions available
            continue

        new_call_depth = call_depth
        if state.history.jumpkind == 'Ijk_Call':
            new_call_depth += 1
        elif state.history.jumpkind == 'Ijk_Ret':
            new_call_depth -= 1

        if self._call_depth is not None and call_depth > self._call_depth:
            l.debug('Do not trace into %s due to the call depth limit', state.ip)
            continue

        new_defs = self._track(state, live_defs, node.irsb.statements if node.irsb is not None else None)

        #corresponding_successors = [n for n in successing_nodes if
        #                            not state.ip.symbolic and n.addr == state.solver.eval(state.ip)]
        #if not corresponding_successors:
        #    continue

        changed = False

        # if every successor can be matched with one or more final states (by IP address),
        # only take over the LiveDefinition of matching states
        if matches:
            add_state_to_sucs = match_state[state]
        else:
            add_state_to_sucs = successing_nodes

        for successing_node in add_state_to_sucs:

            if (state.history.jumpkind == 'Ijk_Call' or state.history.jumpkind.startswith('Ijk_Sys')) and \
                    (state.ip.symbolic or successing_node.addr != state.solver.eval(state.ip)):
                suc_new_defs = self._filter_defs_at_call_sites(new_defs)
            else:
                suc_new_defs = new_defs

            if successing_node in live_defs_per_node:
                defs_for_next_node = live_defs_per_node[successing_node]
            else:
                defs_for_next_node = LiveDefinitions()
                live_defs_per_node[successing_node] = defs_for_next_node

            for var, code_loc_set in suc_new_defs.items():
                # l.debug("Adding %d new definitions for variable %s.", len(code_loc_set), var)
                changed |= defs_for_next_node.add_defs(var, code_loc_set)

        if changed:
            if (self._call_depth is None) or \
                    (self._call_depth is not None and 0 <= new_call_depth <= self._call_depth):
                # Put all reachable successors back to our work-list again
                for successor in self._cfg.get_all_successors(node):
                    nw = DDGJob(successor, new_call_depth)
                    self._worklist_append(nw, worklist, worklist_set)
```

Variable

    <|Reg 16, 8B>

```python
class SimRegisterVariable(SimVariable):

    __slots__ = ['reg', 'size', '_hash']

    def __init__(self, reg_offset, size, ident=None, name=None, region=None, category=None):
        SimVariable.__init__(self, ident=ident, name=name, region=region, category=category)

        self.reg = reg_offset
        self.size = size
        self._hash = None

    def __repr__(self):

        ident_str = "[%s]" % self.ident if self.ident else ""
        region_str = hex(self.region) if isinstance(self.region, int) else self.region

        s = "<%s%s|Reg %s, %sB>" % (region_str, ident_str, self.reg, self.size)

        return s

    def __hash__(self):
        if self._hash is None:
            self._hash = hash(('reg', self.region, self.reg, self.size, self.ident))
        return self._hash

    def __eq__(self, other):
        if isinstance(other, SimRegisterVariable):
            return self.ident == other.ident and \
                   self.reg == other.reg and \
                   self.size == other.size and \
                   self.region == other.region

        return False

class SimVariable:

    __slots__ = ['ident', 'name', 'region', 'category']

    def __init__(self, ident=None, name=None, region=None, category=None):
        """
        :param ident: A unique identifier provided by user or the program. Usually a string.
        :param str name: Name of this variable.
        """
        self.ident = ident
        self.name = name
        self.region = region if region is not None else ""
        self.category = category
```