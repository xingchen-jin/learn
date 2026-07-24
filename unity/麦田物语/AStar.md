## 概念介绍
A* 结合了 **Dijkstra 算法**的准确性和**贪心最佳优先搜索**的速度。  
它通过一个估价函数 `f(n) = g(n) + h(n)` 来评估每个节点，选择“总代价最小”的节点优先探索，从而快速逼近目标。

- **g(n)**：从起点到当前节点 n 的实际代价（已知步数或距离）。
    
- **h(n)**：从 n 到终点的**启发式估计代价**（预测值）。
    
- **f(n)**：总预估代价，即“已经付出的 + 预计还要付出的”。

## 基础数据创建Node & GridNodes


#### 1. 核心数据结构

算法的基础在于如何定义“点”和“图”。

**🔹 Node (节点类)**  
代表地图上的一个格子，实现了 `IComparable` 接口以便在开启列表中进行排序。

- **坐标与状态**:
    - `gridPos`: 网格坐标 (`Vector2Int`)。
    - `isObstacle`: 是否为障碍物 (布尔值)。
    - `parentNode`: 用于回溯路径的父节点引用。
- **寻路代价 (G, H, F)**:
    - `gCost`: 从起点到当前点的实际移动代价。
    - `hCost`: 从当前点到终点的预估代价 (启发式)。
    - `FCost`: 总代价，计算公式为 F=G+HF=G+H 。
- **排序逻辑 (`CompareTo`)**:
    1. 优先比较 `FCost` (越小越优先)。
    2. 若 `FCost` 相同，比较 `hCost` (离终点越近越优先)。

**🔹 GridNodes (网格类)**  
管理所有节点的二维数组。

- **结构**: `Node[,] gridNodes`。
- **功能**: 初始化网格，提供 `GetNode(x, y)` 安全获取节点的方法（包含边界检查）。

---

#### 2. 核心算法流程 (AStar 类)

寻路过程主要分为三个阶段：**初始化** -> **循环查找** -> **路径回溯**。

**第一阶段：构建网格 (`GenerateGridNodes`)**

1. **获取尺寸**: 通过 `GridMapManager` 获取当前场景的网格宽高和原点偏移。
2. **初始化列表**: 创建 `openNodeList` (待检查列表) 和 `closedNodeSet` (已检查集合)。
3. **设置起终点**: 根据传入的 `startPos` 和 `endPos`，结合原点偏移，获取对应的 `Node` 对象。
4. **标记障碍**: 遍历所有网格，查询 `GridMapManager` 中的 `TileDetails`，如果 `isNPCObstacle` 为真，则将该节点标记为 `isObstacle = true`。

**第二阶段：寻找最短路径 (`FindShortestPath`)**  
这是算法的主循环逻辑：

1. **启动**: 将 `startNode` 加入 `openNodeList`。
2. **循环**: 当 `openNodeList` 不为空时：
    - **排序与取出**: 对列表排序，取出索引为 0 的节点作为 `currentNode` (即 F 值最小的节点)。
    - **移入关闭列表**: 从 Open 列表移除，加入 `closedNodeSet`。
    - **终止条件**: 如果 `currentNode == targetNode`，说明找到路径，返回 `true`。
    - **评估邻居**: 调用 `EvaluateNeighbourNodes(currentNode)`。

**第三阶段：评估邻居节点 (`EvaluateNeighbourNodes`)**  
对当前节点周围的 8 个方向 (x: -1~1, y: -1~1) 进行检查：

1. **获取邻居**: 计算邻居坐标，通过 `GetVaildNeighbourNode` 获取节点。
    - _有效性检查_: 坐标不越界、不是障碍物、不在关闭列表中。
2. **处理新邻居**: 如果邻居不在 Open 列表中：
    - **计算 G 值**: `current.gCost + GetDistance(current, neighbour)`。
    - **计算 H 值**: `GetDistance(neighbour, target)`。
    - **设置父节点**: `neighbour.parentNode = current`。
    - **加入 Open 列表**。

> **注意**: 此代码实现中，如果邻居已在 Open 列表中，**没有**执行“检查是否需要更新 G 值”的逻辑（即标准的松弛操作）。这意味着它假设第一次发现的路径即为最优，或者依赖排序机制在后续处理，这在某些复杂地图可能导致非最优解，但在简单网格中通常有效。

---

#### 3. 关键数学逻辑

**🔹 距离计算 (`GetDistance`)**  
使用了 **对角线距离 (Chebyshev / Octile distance)** 的近似算法，而非简单的欧几里得或曼哈顿距离。

- **设定**: 直线移动代价为 10，对角线移动代价为 14 (近似 2×102​×10 )。
- **公式**:  
    设 dx=∣x1−x2∣dx=∣x1​−x2​∣ , dy=∣y1−y2∣dy=∣y1​−y2​∣
    - 若 dx>dydx>dy :

Cost=14×dy+10×(dx−dy)Cost=14×dy+10×(dx−dy)

* 若 dy≥dxdy≥dx :

Cost=14×dx+10×(dy−dx)Cost=14×dx+10×(dy−dx)

**🔹 路径回溯 (`UpdatePathOnMovementStepStack`)**  
一旦找到终点，通过 `parentNode` 链从终点反向遍历回起点：

1. 创建 `MovementStep` 对象。
2. 将网格坐标转换回世界坐标 (加上 `originPoint`)。
3. 压入 `Stack<MovementStep>` 栈中（保证出栈顺序是从起点到终点）。

---

#### 4. 调试与可视化 (AStarTest)

`AStarTest` 脚本用于在 Unity 编辑器中直观地测试寻路效果。

- **触发**: 在 `Update` 中调用 `ShowPathOnGridMap`。
- **显示**:
    - 如果 `displayPath` 开启，调用 `aStar.BuildPath`。
    - 遍历返回的 `movementSteps` 栈。
    - 使用 `Tilemap.SetTile` 在地图上绘制指定的 `displayTile`，从而画出路径。

---

#### 5. 总结与关键点速查

表格

|组件/方法|作用|关键逻辑|
|---|---|---|
|**Node**|数据单元|存储 G/H/F 值，实现 `IComparable` 用于排序|
|**Open List**|待处理列表|使用 `List<Node>`，每次循环需 `Sort()`|
|**Closed Set**|已处理集合|使用 `HashSet<Node>`，防止重复计算|
|**GetDistance**|代价计算|直线 10，对角线 14，避免浮点运算|
|**BuildPath**|入口|协调网格生成、寻路计算和结果输出|

**💡 复习提示**:

- **坐标转换**: 注意代码中频繁出现的 `gridPos` 与 `originPos` 的加减运算，这是为了处理地图原点不在 (0,0) 的情况。
- **性能**: 当前实现使用 `List.Sort()`，在节点极多时性能不如二叉堆 (Binary Heap) 或优先队列，但对于一般 2D 游戏地图足够使用。


