# fast question

## angr.Project　对象如何拿到带符号函数的地址

    None

## angr CFG　剪枝

    None

## 获取　cmp, strcmp等的硬编码字符串

    None

## 符号化命令行参数

```python
    proj = angr.Project(bin_path,load_options={'auto_load_libs':False}, default_analysis_mode='symbolic')
    argv = [proj.filename]
    for i in range(参数个数):
        argv.append(claripy.BVS('arg%d'%(i+1), 字节数*8))
        
    state = proj.factory.entry_state(args=argv)
```