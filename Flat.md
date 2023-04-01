---
title: Deplanarization problem
tags: Reverse
categories: Reverse
top: false
---

## 解题流程

1. 查壳IDA
2. 脚本去平坦化
3. IDA脚本去混淆
4. CRC32识别
5. 解密脚本

## 脚本集

- 去平坦化脚本

  - 命令：

    > python deflat.py attachment 0x400620

  - deflat

    ```python
    #!/usr/bin/env python3
    
    import angr
    import pyvex
    import claripy
    import sys
    sys.path.append('C:\\Users\\du\\Desktop\\CTFTools\\Flat\\am_graph')  ## am_graph路径
    import am_graph
    import struct
    from collections import defaultdict
    
    import logging
    logging.getLogger('angr.state_plugins.symbolic_memory').setLevel(logging.ERROR)
    # logging.getLogger('angr.sim_manager').setLevel(logging.DEBUG)
        
    def get_relevant_nop_nodes(supergraph):
        global pre_dispatcher_node, prologue_node, retn_node
        # relevant_nodes = list(supergraph.predecessors(pre_dispatcher_node))
        relevant_nodes = []
        nop_nodes = []
        for node in supergraph.nodes():
            if supergraph.has_edge(node, pre_dispatcher_node) and node.size > 8:
                # XXX: use node.size is faster than to create a block 
                relevant_nodes.append(node)
                continue
            if node.addr in ( prologue_node.addr, retn_node.addr, pre_dispatcher_node.addr):
                continue
            nop_nodes.append(node)
        return relevant_nodes, nop_nodes
    
    
    def symbolic_execution(start_addr, hook_addr=None, modify=None, inspect=False):
        
        def retn_procedure(state):
            global project
            ip = state.se.eval(state.regs.ip)
            project.unhook(ip)
            return
        
        def statement_inspect(state):
            global modify_value
            expressions = list(state.scratch.irsb.statements[state.inspect.statement].expressions)
            if len(expressions) != 0 and isinstance(expressions[0], pyvex.expr.ITE):
                state.scratch.temps[expressions[0].cond.tmp] = modify_value
                state.inspect._breakpoints['statement'] = []
    
        global project, relevant_block_addrs, modify_value
        if hook_addr != None:
            project.hook(hook_addr, retn_procedure, length=5)
        if modify != None:
            modify_value = modify
        state = project.factory.blank_state(addr=start_addr, remove_options={angr.sim_options.LAZY_SOLVES})
        if inspect:
            state.inspect.b('statement', when=angr.state_plugins.inspect.BP_BEFORE, action=statement_inspect)
        sm = project.factory.simulation_manager(state)
        sm.step()
        while len(sm.active) > 0:
            for active_state in sm.active:
                if active_state.addr in relevant_block_addrs:
                    return active_state.addr
            sm.step()
    
    def fill_nop(data, start_addr, length):
        global opcode
        for i in range(0, length):
            data[start_addr + i] = ord(opcode['nop'])
    
    def fill_jmp_offset(data, start, offset):
        jmp_offset = struct.pack('<i', offset)  # bytes
        for i in range(4):
            data[start + i] = jmp_offset[i]
        
    def patch_byte(data, offset, value):
        # operate on bytearray, not str
        data[offset] = ord(value)
    
    if __name__ == '__main__':
        if len(sys.argv) != 3:
            print('Usage: python deflat.py filename function_address(hex)')
            exit(0)
        
        opcode = {'a':'\x87', 'ae': '\x83', 'b':'\x82', 'be':'\x86', 'c':'\x82', 'e':'\x84', 'z':'\x84', 'g':'\x8F', 
                  'ge':'\x8D', 'l':'\x8C', 'le':'\x8E', 'na':'\x86', 'nae':'\x82', 'nb':'\x83', 'nbe':'\x87', 'nc':'\x83',
                  'ne':'\x85', 'ng':'\x8E', 'nge':'\x8C', 'nl':'\x8D', 'nle':'\x8F', 'no':'\x81', 'np':'\x8B', 'ns':'\x89',
                  'nz':'\x85', 'o':'\x80', 'p':'\x8A', 'pe':'\x8A', 'po':'\x8B', 's':'\x88', 'nop':'\x90', 'jmp':'\xE9', 'j':'\x0F'}
        
        filename = sys.argv[1]
        start = int(sys.argv[2], 16)
    
        project = angr.Project(filename, load_options={'auto_load_libs': False})
        cfg = project.analyses.CFGFast(normalize=True)  # do normalize to avoid overlapping blocks
        target_function = cfg.functions.get(start)
        # A super transition graph is a graph that looks like IDA Pro's CFG
        supergraph = am_graph.to_supergraph(target_function.transition_graph)
    
        base_addr = project.loader.main_object.mapped_base >> 12 << 12
    
        # get prologue_node and retn_node
        prologue_node = None
        for node in supergraph.nodes():
            if supergraph.in_degree(node) == 0:
                prologue_node = node
            if supergraph.out_degree(node) == 0:
                retn_node = node
        
        if prologue_node is None or prologue_node.addr != start:
            print("Something must be wrong...")
            sys.exit(-1)
        
        main_dispatcher_node = list(supergraph.successors(prologue_node))[0]
        for node in supergraph.predecessors(main_dispatcher_node):
            if node.addr != prologue_node.addr:
                pre_dispatcher_node = node
                break
        
        relevant_nodes, nop_nodes = get_relevant_nop_nodes(supergraph)
        print('*******************relevant blocks************************')
        print('prologue: %#x' % start)
        print('main_dispatcher: %#x' % main_dispatcher_node.addr)
        print('pre_dispatcher: %#x' % pre_dispatcher_node.addr)
        print('retn: %#x' % retn_node.addr)
        relevant_block_addrs = [node.addr for node in relevant_nodes]
        print('relevant_blocks:', [hex(addr) for addr in relevant_block_addrs])
    
        print('*******************symbolic execution*********************')
        relevants = relevant_nodes
        relevants.append(prologue_node)
        relevants_without_retn = list(relevants)
        relevants.append(retn_node)
        relevant_block_addrs.extend([prologue_node.addr, retn_node.addr])
    
        flow = defaultdict(list)
        modify_value = None
        patch_instrs = {}
        for relevant in relevants_without_retn:
            print('-------------------dse %#x---------------------' % relevant.addr)
            block = project.factory.block(relevant.addr, size=relevant.size)
            has_branches = False
            hook_addr = None
            for ins in block.capstone.insns:
                if ins.insn.mnemonic.startswith('cmov'):
                    patch_instrs[relevant] = ins
                    has_branches = True
                elif ins.insn.mnemonic.startswith('call'):
                    hook_addr = ins.insn.address
            if has_branches:
                flow[relevant].append(symbolic_execution(relevant.addr, hook_addr, claripy.BVV(1, 1), True))
                flow[relevant].append(symbolic_execution(relevant.addr, hook_addr, claripy.BVV(0, 1), True))
            else:
                flow[relevant].append(symbolic_execution(relevant.addr, hook_addr))
                
        print('************************flow******************************')
        for k, v in flow.items():
            print('%#x: ' % k.addr, [hex(child) for child in v])
        
        # print retn_node flow. Actually, it's [].
        print('%#x: ' % retn_node.addr, [])
    
        print('************************patch*****************************')
        with open(filename, 'rb') as origin:
            # Attention: can't transform to str by calling decode() directly. so use bytearray instead.
            origin_data = bytearray(origin.read())
            origin_data_len = len(origin_data)
    
        recovery_file = filename + '_recovered'
        recovery = open(recovery_file, 'wb')
    
        # patch irrelevant blocks
        for nop_node in nop_nodes:
            fill_nop(origin_data, nop_node.addr - base_addr, nop_node.size)
        
        # remove unnecessary control flows
        for parent, childs in flow.items():
            if len(childs) == 1:
                parent_block = project.factory.block(parent.addr, size=parent.size)
                last_instr = parent_block.capstone.insns[-1]
                file_offset = last_instr.address - base_addr
                # patch the last jmp instruction
                patch_byte(origin_data, file_offset, opcode['jmp'])
                file_offset += 1
                fill_nop(origin_data, file_offset, last_instr.size - 1)
                fill_jmp_offset(origin_data, file_offset, childs[0] - last_instr.address - 5)
            else:
                instr = patch_instrs[parent]
                file_offset = instr.address - base_addr
                # patch instructions starting from `cmovx` to the end of block
                fill_nop(origin_data, file_offset, parent.addr + parent.size - base_addr - file_offset)
                patch_byte(origin_data, file_offset, opcode['j'])
                patch_byte(origin_data, file_offset + 1, opcode[instr.mnemonic[4:]])
                fill_jmp_offset(origin_data, file_offset + 2, childs[0] - instr.address - 6)
                file_offset += 6
                patch_byte(origin_data, file_offset, opcode['jmp'])
                fill_jmp_offset(origin_data, file_offset + 1, childs[1] - (instr.address + 6) - 5)
        
        assert len(origin_data) == origin_data_len, "Error: size of data changed!!!"
        recovery.write(origin_data)
        recovery.close()
        print('Successful! The recovered file: %s' % recovery_file)
    ```

  - am_graph

    ```python
    import itertools
    from collections import defaultdict
    
    import networkx
    
    from angr.knowledge_plugins import Function
    
    
    def grouper(iterable, n, fillvalue=None):
        "Collect data into fixed-length chunks or blocks"
        args = [iter(iterable)] * n
        return itertools.izip_longest(*args, fillvalue=fillvalue)
    
    
    def to_supergraph(transition_graph):
        """
        Convert transition graph of a function to a super transition graph. A super transition graph is a graph that looks
        like IDA Pro's CFG, where calls to returning functions do not terminate basic blocks.
    
        :param networkx.DiGraph transition_graph: The transition graph.
        :return: A converted super transition graph
        :rtype networkx.DiGraph
        """
    
        # make a copy of the graph
        transition_graph = networkx.DiGraph(transition_graph)
    
        # remove all edges that transitions to outside
        for src, dst, data in list(transition_graph.edges(data=True)):
            if data['type'] in ('transition', 'exception') and data.get('outside', False) is True:
                transition_graph.remove_edge(src, dst)
            if transition_graph.in_degree(dst) == 0:
                transition_graph.remove_node(dst)
    
        edges_to_shrink = set()
    
        # Find all edges to remove in the super graph
        for src in transition_graph.nodes():
            edges = transition_graph[src]
    
            # there are two types of edges we want to remove:
            # - call or fakerets, since we do not want blocks to break at calls
            # - boring jumps that directly transfer the control to the block immediately after the current block. this is
            #   usually caused by how VEX breaks down basic blocks, which happens very often in MIPS
    
    
    
            if len(edges) == 1 and src.addr + src.size == next(iter(edges.keys())).addr:
                dst = next(iter(edges.keys()))
                dst_in_edges = transition_graph.in_edges(dst)
                if len(dst_in_edges) == 1:
                    edges_to_shrink.add((src, dst))
                    continue
    
            if any(iter('type' in data and data['type'] not in ('fake_return', 'call') for data in edges.values())):
                continue
    
            for dst, data in edges.items():
                if isinstance(dst, Function):
                    continue
                if 'type' in data and data['type'] == 'fake_return':
                    if all(iter('type' in data and data['type'] in ('fake_return', 'return_from_call')
                                for _, _, data in transition_graph.in_edges(dst, data=True))):
                        edges_to_shrink.add((src, dst))
                    break
    
        # Create the super graph
        super_graph = networkx.DiGraph()
    
        supernodes_map = {}
    
        function_nodes = set()  # it will be traversed after all other nodes are added into the supergraph
    
        for node in transition_graph.nodes():
    
            if isinstance(node, Function):
                function_nodes.add(node)
                # don't put functions into the supergraph
                continue
    
            dests_and_data = transition_graph[node]
    
            # make a super node
            if node in supernodes_map:
                src_supernode = supernodes_map[node]
            else:
                src_supernode = SuperCFGNode.from_cfgnode(node)
                supernodes_map[node] = src_supernode
                # insert it into the graph
                super_graph.add_node(src_supernode)
    
            if not dests_and_data:
                # might be an isolated node
                continue
    
            # Take src_supernode off the graph since we might modify it
            if src_supernode in super_graph:
                existing_in_edges = list(super_graph.in_edges(src_supernode, data=True))
                existing_out_edges = list(super_graph.out_edges(src_supernode, data=True))
                super_graph.remove_node(src_supernode)
            else:
                existing_in_edges = [ ]
                existing_out_edges = [ ]
    
            for dst, data in dests_and_data.items():
    
                edge = (node, dst)
    
                if edge in edges_to_shrink:
    
                    if dst in supernodes_map:
                        dst_supernode = supernodes_map[dst]
                    else:
                        dst_supernode = None
    
                    src_supernode.insert_cfgnode(dst)
    
                    # update supernodes map
                    supernodes_map[dst] = src_supernode
    
                    # merge the other supernode
                    if dst_supernode is not None:
                        src_supernode.merge(dst_supernode)
    
                        for src in dst_supernode.cfg_nodes:
                            supernodes_map[src] = src_supernode
    
                        # link all out edges of dst_supernode to src_supernode
                        for dst_, data_ in super_graph[dst_supernode].items():
                            super_graph.add_edge(src_supernode, dst_, **data_)
    
                        # link all in edges of dst_supernode to src_supernode
                        for src_, _, data_ in super_graph.in_edges(dst_supernode, data=True):
                            super_graph.add_edge(src_, src_supernode, **data_)
    
                            if 'type' in data_ and data_['type'] in ('transition', 'exception'):
                                if not ('ins_addr' in data_ and 'stmt_idx' in data_):
                                    # this is a hack to work around the issue in Function.normalize() where ins_addr and
                                    # stmt_idx weren't properly set onto edges
                                    continue
                                src_supernode.register_out_branch(data_['ins_addr'], data_['stmt_idx'], data_['type'],
                                                                  dst_supernode.addr
                                                                  )
    
                        super_graph.remove_node(dst_supernode)
    
                else:
                    if isinstance(dst, Function):
                        # skip all functions
                        continue
    
                    # make a super node
                    if dst in supernodes_map:
                        dst_supernode = supernodes_map[dst]
                    else:
                        dst_supernode = SuperCFGNode.from_cfgnode(dst)
                        supernodes_map[dst] = dst_supernode
    
                    super_graph.add_edge(src_supernode, dst_supernode, **data)
    
                    if 'type' in data and data['type'] in ('transition', 'exception'):
                        if not ('ins_addr' in data and 'stmt_idx' in data):
                            # this is a hack to work around the issue in Function.normalize() where ins_addr and
                            # stmt_idx weren't properly set onto edges
                            continue
                        src_supernode.register_out_branch(data['ins_addr'], data['stmt_idx'], data['type'],
                                                          dst_supernode.addr
                                                          )
    
            # add back the node (in case there are no edges)
            super_graph.add_node(src_supernode)
            # add back the old edges
            for src, _, data in existing_in_edges:
                super_graph.add_edge(src, src_supernode, **data)
            for _, dst, data in existing_out_edges:
                super_graph.add_edge(src_supernode, dst, **data)
    
        for node in function_nodes:
            in_edges = transition_graph.in_edges(node, data=True)
    
            for src, _, data in in_edges:
                if not ('ins_addr' in data and 'stmt_idx' in data):
                    # this is a hack to work around the issue in Function.normalize() where ins_addr and
                    # stmt_idx weren't properly set onto edges
                    continue
                supernode = supernodes_map[src]
                supernode.register_out_branch(data['ins_addr'], data['stmt_idx'], data['type'], node.addr)
    
        return super_graph
    
    
    class OutBranch:
        def __init__(self, ins_addr, stmt_idx, branch_type):
            self.ins_addr = ins_addr
            self.stmt_idx = stmt_idx
            self.type = branch_type
    
            self.targets = set()
    
        def __repr__(self):
            if self.ins_addr is None:
                return "<OutBranch at None, type %s>" % self.type
            return "<OutBranch at %#x, type %s>" % (self.ins_addr, self.type)
    
        def add_target(self, addr):
            self.targets.add(addr)
    
        def merge(self, other):
            """
            Merge with the other OutBranch descriptor.
    
            :param OutBranch other: The other item to merge with.
            :return: None
            """
    
            assert self.ins_addr == other.ins_addr
            assert self.type == other.type
    
            o = self.copy()
            o.targets |= other.targets
    
            return o
    
        def copy(self):
            o = OutBranch(self.ins_addr, self.stmt_idx, self.type)
            o.targets = self.targets.copy()
            return o
    
        def __eq__(self, other):
            if not isinstance(other, OutBranch):
                return False
    
            return self.ins_addr == other.ins_addr and \
                   self.stmt_idx == other.stmt_idx and \
                   self.type == other.type and \
                   self.targets == other.targets
    
        def __hash__(self):
            return hash((self.ins_addr, self.stmt_idx, self.type))
    
    
    class SuperCFGNode:
        def __init__(self, addr):
            self.addr = addr
    
            self.cfg_nodes = [ ]
    
            self.out_branches = defaultdict(dict)
    
        @property
        def size(self):
            return sum(node.size for node in self.cfg_nodes)
    
        @classmethod
        def from_cfgnode(cls, cfg_node):
            s = cls(cfg_node.addr)
    
            s.cfg_nodes.append(cfg_node)
    
            return s
    
        def insert_cfgnode(self, cfg_node):
            # TODO: Make it binary search/insertion
            for i, n in enumerate(self.cfg_nodes):
                if cfg_node.addr < n.addr:
                    # insert before n
                    self.cfg_nodes.insert(i, cfg_node)
                    break
                elif cfg_node.addr == n.addr:
                    break
            else:
                self.cfg_nodes.append(cfg_node)
    
            # update addr
            self.addr = self.cfg_nodes[0].addr
    
        def register_out_branch(self, ins_addr, stmt_idx, branch_type, target_addr):
            if ins_addr not in self.out_branches or stmt_idx not in self.out_branches[ins_addr]:
                self.out_branches[ins_addr][stmt_idx] = OutBranch(ins_addr, stmt_idx, branch_type)
    
            self.out_branches[ins_addr][stmt_idx].add_target(target_addr)
    
        def merge(self, other):
            """
            Merge another supernode into the current one.
    
            :param SuperCFGNode other: The supernode to merge with.
            :return: None
            """
    
            for n in other.cfg_nodes:
                self.insert_cfgnode(n)
    
            for ins_addr, outs in other.out_branches.items():
                if ins_addr in self.out_branches:
                    for stmt_idx, item in outs.items():
                        if stmt_idx in self.out_branches[ins_addr]:
                            self.out_branches[ins_addr][stmt_idx].merge(item)
                        else:
                            self.out_branches[ins_addr][stmt_idx] = item
    
                else:
                    item = next(iter(outs.values()))
                    self.out_branches[ins_addr][item.stmt_idx] = item
    
        def __repr__(self):
            return "<SuperCFGNode %#08x, %d blocks, %d out branches>" % (self.addr, len(self.cfg_nodes),
                                                                         len(self.out_branches)
                                                                         )
    
        def __hash__(self):
            return hash(('supercfgnode', self.addr))
    
        def __eq__(self, other):
            if not isinstance(other, SuperCFGNode):
                return False
    
            return self.addr == other.addr
    ```

- 去混淆脚本

  ```python
  st = 0x0000000000400620 
  end = 0x0000000000402144 
   
  def next_instr(addr):
      return addr+idc.get_item_size(addr)		#获取指令或数据长度，这个函数的作用就是去往下一条指令
      
  addr = st
  while(addr<end):
      next = next_instr(addr)
      if "ds:dword_603054" in GetDisasm(addr):	#GetDisasm(addr)得到addr的反汇编语句
          while(True):
              addr = next
              next = next_instr(addr)
              if "jnz" in GetDisasm(addr):
                  dest = idc.get_operand_value(addr, 0)		#得到操作数，就是指令后的数
                  ida_bytes.patch_byte(addr, 0xe9)     #0xe9 jmp后面的四个字节是偏移
                  ida_bytes.patch_byte(addr+5, 0x90)   #nop第五个字节
                  offset = dest - (addr + 5)  #调整为正确的偏移地址 也就是相对偏移地址 - 当前指令后的地址
                  ida_bytes.patch_dword(addr + 1, offset) #把地址赋值给jmp后
                  print("patch bcf: 0x%x"%addr)
                  addr = next
                  break
      else:
          addr = next
  ```

- 解密脚本

  ```python
  secret = [0xBC8FF26D43536296, 0x520100780530EE16, 0x4DC0B5EA935F08EC,
            0x342B90AFD853F450, 0x8B250EBCAA2C3681, 0x55759F81A2C68AE4]
  key = 0xB0004B7679FA26B3
  
  flag = ""
  
  # 产生CRC32查表法所用的表
  for s in secret:
      for i in range(64):
          sign = s & 1                   #判断首位正负
          if sign == 1:
              s ^= key
          s //= 2
          if sign == 1:
              s |= 0x8000000000000000    # 防止负值除2，溢出为正值
      print(hex(s))                      # 输出表
      # 计算CRC64
      j = 0
      while j < 8:
          flag += chr(s&0xFF)
          s >>= 8
          j += 1
  print(flag)
  ```