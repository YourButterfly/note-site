# example

## 包含返回常量语句的基本块

``` ql
class BasicBlockWithReturn extends BasicBlock {
  BasicBlockWithReturn() {
    exists (Literal l|
      this.contains(any(ReturnStmt r| r.getExpr()=l))
    )
  }
}
```
