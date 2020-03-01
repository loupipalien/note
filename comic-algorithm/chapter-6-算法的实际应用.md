### 算法的实际应用

#### BitMap 的巧用
TODO

#### LRU 算法的应用
TODO

#### A 星寻路算法
##### 两个集合和一个公式
- OpenList: 可达到的格子
- CloseList: 已到达的格子
- F = G + H
  - G: 从起点走到当前格子的成本, 也就是花费了多少步
  - H: 在不考虑障碍的情况下, 从当前格子走到目标格子的距离, 也就是离目标还有多远
  - F: G 和 H 的综合评估, 也就是从起点到达当前格子, 再从当前格子到达目标格子的总步数

##### 代码实现
```Java
public class Solution {

    public static void main(String[] args) {
        int[][] maze = {
                {0, 0, 0, 0, 0, 0, 0},
                {0, 0, 0, 1, 0, 0, 0},
                {0, 0, 0, 1, 0, 0, 0},
                {0, 0, 0, 1, 0, 0, 0},
                {0, 0, 0, 0, 0, 0, 0},
        };
        Grid start = new Grid(2, 1);
        Grid end = new Grid(2, 5);
        // 搜索迷宫终点
        Grid grid = aStarSearch(maze, start, end);
        // 回溯迷宫路径
        List<Grid> path = new ArrayList<>();
        while (grid != null) {
            path.add(new Grid(grid.x, grid.y));
            grid = grid.parent;
        }
        // 输出迷宫路径, 用 * 表示
        for (int i = 0; i < maze.length; i++) {
            for (int j = 0; j < maze[0].length; j++) {
                if (contains(path, i, j)) {
                    System.out.print("*, ");
                } else {
                    System.out.print(maze[i][j] + ", ");
                }
            }
            System.out.println();
        }
    }

    public static Grid aStarSearch(int[][] maze, Grid start, Grid end) {
        List<Grid> openList = new ArrayList<>();
        List<Grid> closeList = new ArrayList<>();
        // 把起点加入 openList;
        openList.add(start);
        // 循环检查当前方格节点
        while (openList.size() > 0) {
            // 在 openList 中查找 F 值最小的点, 将其当做当前方格节点
            Grid curr = findMin(openList);
            // 将当前节点从 openList 移除添加到 closeList 中
            openList.remove(curr);
            closeList.add(curr);
            // 找到当前节点所有邻居的节点
            List<Grid> neighbors = findNeighbors(maze, curr, openList, closeList);
            // 将邻居节点初始化后放入 openList 中
            for (Grid neighbor : neighbors) {
                neighbor.initGrid(curr, end);
                openList.add(neighbor);
            }
            // 如果终点在 openList 中, 直接返回终点格子
            for (Grid grid : openList) {
                if (grid.x == end.x && grid.y == end.y) return grid;
            }
        }
        // 遍历可到达的所有节点后仍然找不到终点, 说明终点不可达
        return null;
    }

    private static Grid findMin(List<Grid> openList) {
        Grid min = openList.get(0);
        for (Grid grid : openList) {
            if (grid.f < min.f) {
                min = grid;
            }
        }
        return min;
    }

    private static List<Grid> findNeighbors(int[][] maze, Grid grid, List<Grid> openList, List<Grid> closeList) {
        List<Grid> neighbors = new ArrayList<>();
        if (isValidGrid(maze, grid.x, grid.y - 1, openList, closeList)) {
            neighbors.add(new Grid(grid.x, grid.y - 1));
        }
        if (isValidGrid(maze, grid.x, grid.y + 1, openList, closeList)) {
            neighbors.add(new Grid(grid.x, grid.y + 1));
        }
        if (isValidGrid(maze, grid.x - 1, grid.y, openList, closeList)) {
            neighbors.add(new Grid(grid.x - 1, grid.y));
        }
        if (isValidGrid(maze, grid.x + 1, grid.y, openList, closeList)) {
            neighbors.add(new Grid(grid.x + 1, grid.y));
        }
        return neighbors;
    }

    private static boolean isValidGrid(int[][] maze, int x, int y, List<Grid> openList, List<Grid> closeList) {
        // 是否越界
        if (x < 0 || x >= maze.length || y < 0 ||y >= maze[0].length) return false;
        // 是否有障碍物
        if (maze[x][y] == 1) return false;
        // 是否已经在 openList 中
        if (contains(openList, x, y)) return false;
        // 是否在 closeList 中
        if (contains(closeList, x, y)) return false;

        return true;
    }

    private static boolean contains(List<Grid> grids, int x, int y) {
        for (Grid grid : grids) {
            if (grid.x == x && grid.y == y) return true;
        }
        return false;
    }

    public static class Grid {
        public int x;
        public int y;
        public int f;
        public int g;
        public int h;
        public Grid parent;

        public Grid(int x, int y) {
            this.x = x;
            this.y = y;
        }

        public void initGrid(Grid parent, Grid end) {
            this.parent = parent;
            if (parent != null) {
                this.g = parent.g + 1;
            } else {
                this.g = 1;
            }
            this.h = Math.abs(this.x - end.x) + Math.abs(this.y - end.y);
            this.f = this.g + this.h;
        }
    }
}
```

#### 如何实现红包算法
TODO
