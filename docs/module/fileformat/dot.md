# DOT 语言

Dot 语言的抽象语法

```txt
        graph:[ strict ] (graph | digraph) [ ID ] '{' stmt_list '}'
    stmt_list:[ stmt [ ';' ] stmt_list ]
         stmt:node_stmt
             |edge_stmt
             |attr_stmt
             |ID '=' ID
             |subgraph
    attr_stmt:(graph | node | edge) attr_list
    attr_list:'[' [ a_list ] ']' [ attr_list ]
       a_list:ID '=' ID [ (';' | ',') ] [ a_list ]
    edge_stmt:(node_id | subgraph) edgeRHS [ attr_list ]
      edgeRHS:edgeop (node_id | subgraph) [ edgeRHS ]
    node_stmt:node_id [ attr_list ]
      node_id:ID [ port ]
         port:':' ID [ ':' compass_pt ]
             |':' compass_pt
     subgraph:[ subgraph [ ID ] ] '{' stmt_list '}'
   compass_pt:(n | ne | e | se | s | sw | w | nw | c | _)
```

关键字node，edge，graph，digraph，subgraph和strict与大小写无关
'('和')'表示必选组，如(graph | digraph)'表示graph和digraph两者有且只有其一
单引号表示内容是字符，如'='便是字符'='，而不是赋值
