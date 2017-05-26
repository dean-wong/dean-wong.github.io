---
layout: post
title:  "Jupyter 小技巧"
date:   2017-04-25
categories: dean note
---
## 在Jupyter中画dot图

```python
# 用inline的方式绘dot图
from nxpd import nxpdParams
nxpdParams['show'] = 'ipynb'
```

```python
import networkx as nx
from nxpd import draw
G = nx.DiGraph()
G.graph['rankdir'] = 'LR'
# G.graph['dpi'] = 120
G.add_cycle(range(4))
G.add_node(0, color='red', style='filled', fillcolor='pink')
G.add_node(1, shape='square')
G.add_node(3, style='filled', fillcolor='#00ffff')
G.add_edge(0, 1, color='red', style='dashed')
G.add_edge(3, 3, label='a')
draw(G)
```

```python
C1 = nx.drawing.nx_pydot.read_dot('tree.dot')
draw(C1)
```

