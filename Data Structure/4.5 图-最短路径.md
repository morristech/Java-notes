# 图

## 5、最短路径

**单点最短路径**：给定一幅加权有向和一个起点s，计算从s到给定的目的顶点v的权重之和最小的路径。

**性质**：

1. 路径是有向的
2. 权重不一定等价于距离（也可以是其他的某种花费，所以可以将最短路径应用于其他场景）
3. 并不是所有的顶点都是可达的
4. 负权重会使问题更加复杂
5. 最短路径一般都是简单的
6. 最短路径不一定是唯一的
7. 可能存在平行环和自环

### 5.1 加权有向图的数据结构

#### 5.1.1 加权有向图的边

	public class DirectedEdge { 
	    private final int v;          // 边的起点
	    private final int w;          // 边的终点
	    private final double weight;  // 边的权重
	
	    public DirectedEdge(int v, int w, double weight) {
	        this.v = v;
	        this.w = w;
	        this.weight = weight;
	    }
	
	    public int from() { return v; }
	
	    public int to() { return w; }
	
	    public double weight() { return weight; }
	}

#### 5.1.2 加权有向图

以上是加权有向图的边的定义。我们可以利用上述的边，来实现加权有向图的定义：

	public class EdgeWeightedDigraph {
	    private final int V;                // 顶点的总数
	    private int E;                      // 边的总数
	    private Bag<DirectedEdge>[] adj;    // 邻接表
	    
	    public EdgeWeightedDigraph(int V) {
	        this.V = V;
	        this.E = 0;
	        this.indegree = new int[V];
	        adj = (Bag<DirectedEdge>[]) new Bag[V];
	        for (int v = 0; v < V; v++)
	            adj[v] = new Bag<DirectedEdge>();
	    }
	
	    public void addEdge(DirectedEdge e) {
	        adj[e.from()].add(e);
	        E++;
	    }
	
        public Iterable<DirectedEdge> adj(int v) {
            return adj[v];
        }

	    public int V() { return V; }
	
	    public int E() { return E; }
	}

#### 5.1.3 最短路的API

	public class SP {
	    private double[] distTo;          // distTo[v] 代表 s->v 的最短路径
	    private DirectedEdge[] edgeTo;    // edgeTo[v] 代表 s->v 路径的最后一条边，
                                          // 我们可以根据这条边来得到从起点到目的顶点的最短路径
	
	    public DijkstraSP(EdgeWeightedDigraph G, int s) {
	        distTo = new double[G.V()];
	        edgeTo = new DirectedEdge[G.V()];
	
	        for (int v = 0; v < G.V(); v++)
	            distTo[v] = Double.POSITIVE_INFINITY;
	        distTo[s] = 0.0;
	    }
	
        // 从顶点 s 到 v 的距离，无穷大代表不存在
	    public double distTo(int v) {
	        return distTo[v];
	    }
	
	    public boolean hasPathTo(int v) {
	        return distTo[v] < Double.POSITIVE_INFINITY;
	    }
	
        // 获取从s到v的最短路径
	    public Iterable<DirectedEdge> pathTo(int v) {
	        if (!hasPathTo(v)) return null;
	        Stack<DirectedEdge> path = new Stack<DirectedEdge>();
	        for (DirectedEdge e = edgeTo[v]; e != null; e = edgeTo[e.from()]) {
	            path.push(e);
	        }
	        return path;
	    }
	}

#### 5.1.4 边的松弛

以下是边的松弛的算法，

	private void relax(DirectedEdge e) {
	    int v = e.from(), w = e.to();
	    if (distTo[w] > distTo[v] + e.weight()) {
	        distTo[w] = distTo[v] + e.weight();
	        edgeTo[w] = e;
	    }
	}

我们可以结合下图来理解边的松弛的意义：

![](http://chuantu.biz/t6/111/1508917448x1942832654.png)

所以边的松弛就是：是s->w的路径比较短，还是从s->v再v->w比较短。注意这里的s->w表示的是一个s到w的路径，s->v是s到v的路径，而v->w是一条边。

#### 5.1.5 顶点的松弛

和上面的边的松弛类似，下面是顶点的松弛：

![](http://chuantu.biz/t6/111/1508917524x1942832654.png)

	private void relax(EdgeWeightedDigraph G, int v) {
         for(DirectedEdge e : G.adj(v)) {
    	    int w = e.to();
		    if (distTo[w] > distTo[v] + e.weight()) {
		        distTo[w] = distTo[v] + e.weight();
		        edgeTo[w] = e;
		    }
         }
	}

其实这里相当于对与v相邻的全部的边进行松弛：判断是s->w1, s->w2, s->w3...比较短，还是s->v再v->w1, v->w2, v-w3...比较短。(w1, w2, w3...表示以v为起点的所有的边的终点)

### 5.2 最短路径

#### 5.2.1 理论基础

**最短路径的最优性条件**：令G为一幅加权有向图，顶点s是G中的起点，distTo[]是一个由顶点索引的数组，保存的是G中路径的长度。对于从s可达的所有顶点v，distTo[v]的值是从s到v的某条路径的长度，对于从s不可达的所有顶点v，该值为无穷大。当且仅当对于从v到w的任意一条边e，这些值满足distTo[w]<=distTo[v]+e.weight()时，它们是最短路径的长度。

**通用最短路径算法**：将distTo[s]初始化为0，其他distTo[]元素初始化为无穷大，继续如下操作：放松G中的任意边，直到存在有效边为止。对于任意从s可达的顶点w，在进行这些操作之后，distTo[w]的值即为从s到w的最短路径的长度（且edgetT[w]的值即为该路径上的最后一条边）。

#### 5.2.2 Dijkstra算法

![](http://chuantu.biz/t6/111/1508919179x1942832654.png)

我们可以这么理解Dijkstra算法. 我们从v0开始使用广度优先对图中的每个顶点进行遍历，对与每个顶点相连的边进行松弛:

1. 首先，对v1进行松弛：
	1. 对v0->v1，显然最短路径是从v0->v1，长度是1；
	2. v0->v2长度是5，v0->v1->v2长度是4，所以v0->v2长度置为4；
	3. v0->v3长度是无穷大，v0->v1->v3长度是8，所以v0->v3长度置为8；
	4. v0->v4长度是无穷大，v0->v1->v4长度是6，所以v0->v4长度置为6；
2. 再对v2进行松弛：
	1. v0->v1长度是1，v0->v2->v1长度是8，所以v0->v1长度置为1；
	2. v0->v4长度是6，v0->v2->v4长度是5，所以v0->v4长度置为5；
	3. v0->v5长度是无穷大，v0->v2->v5长度是6，所以v0->v5长度置为6；
3. ......依次类推，直到终点

但是上面的思路有一个问题，就是如果我们从V2到v1开始进行松弛，那么到V2的距离会被先置为5，然后用v0->v2等于5来更新v4,v5,v1. 然后再用v1，更新v0->v2为4，这时候v4,v5就得不到更新了. 所以，v0到一个点的最短路径被更新的时候，我们需要将其再次加入到队列中对其进行计算。

下面是使用广度优先来实现的Dijkstra算法：

	public class Dijkstra {
	    private double[] distTo;
	    private DirectedEdge[] edgeTo;
	    private LinkedList<Integer> queue;
	
	    public Dijkstra(EdgeWeightedDigraph G, int s) {
	        // 初始化数据
	        distTo = new double[G.V()];
	        edgeTo = new DirectedEdge[G.V()];
	
            // 注意指定的起始点与其他点相连的边不为无穷大，需要对其进行处理
	        distTo[s] = 0.0;
	        edgeTo[s] = new DirectedEdge(s, s, 0);
	        for (int v = 0; v < G.V(); v++) {
	            distTo[v] = Double.POSITIVE_INFINITY;
	        }
	        for (DirectedEdge edge : G.adj(s)) {
	            distTo[edge.to()] = edge.weight();
	            edgeTo[edge.to()] = edge;
	        }
	
	        // 用于广度优先
	        boolean[] visited = new boolean[G.V()]; // 标记指定的点是否被访问过
	        queue = new LinkedList<Integer>();
	        queue.add(s); // 先将起始点加入到集合中
	
	        while (!queue.isEmpty()) {
                // 从队列中弹出一个记录
	            int current = queue.remove(0);
                // 指定的记录可能会被重复添加到集合中，对于重复添加的直接跳过
	            if (visited[current]) continue;
                // 标记当前节点为”已访问“
	            visited[current] = true;
                // 对所有与当前的结点相连的边进行遍历
	            for (DirectedEdge e : G.adj(current)) {
	                int v = e.from(), w = e.to();
                    // 如果指定的点还没有被访问过，就将其加入到集合中
	                if (!visited[w]) queue.add(w);
                    // 对边进行松弛
	                if (distTo[w] > distTo[v] + e.weight()) {
	                    distTo[w] = distTo[v] + e.weight();
	                    edgeTo[w] = e;
                        if (visited[w]) { // 值已经被更新，重新加入进去计算
                            queue.add(w);
                            visited[w] = false;
                        }
	                }
	            }
	        }
	    }
	
        // 构造数据并进行测试
	    public static void main(String ...args) {
	        EdgeWeightedDigraph digraph = new EdgeWeightedDigraph(9);
	        digraph.addEdge(new DirectedEdge(0, 1, 1));
	        digraph.addEdge(new DirectedEdge(0, 2, 5));
	        digraph.addEdge(new DirectedEdge(1, 0, 1));
	        digraph.addEdge(new DirectedEdge(1, 2, 3));
	        digraph.addEdge(new DirectedEdge(1, 3, 7));
	        digraph.addEdge(new DirectedEdge(1, 4, 5));
	        digraph.addEdge(new DirectedEdge(2, 0, 5));
	        digraph.addEdge(new DirectedEdge(2, 1, 3));
	        digraph.addEdge(new DirectedEdge(2, 4, 1));
	        digraph.addEdge(new DirectedEdge(2, 5, 7));
	        digraph.addEdge(new DirectedEdge(3, 1, 7));
	        digraph.addEdge(new DirectedEdge(3, 4, 2));
	        digraph.addEdge(new DirectedEdge(3, 6, 3));
	        digraph.addEdge(new DirectedEdge(4, 1, 5));
	        digraph.addEdge(new DirectedEdge(4, 2, 1));
	        digraph.addEdge(new DirectedEdge(4, 3, 2));
	        digraph.addEdge(new DirectedEdge(4, 5, 3));
	        digraph.addEdge(new DirectedEdge(4, 6, 6));
	        digraph.addEdge(new DirectedEdge(4, 7, 9));
	        digraph.addEdge(new DirectedEdge(5, 2, 7));
	        digraph.addEdge(new DirectedEdge(5, 4, 3));
	        digraph.addEdge(new DirectedEdge(5, 7, 5));
	        digraph.addEdge(new DirectedEdge(6, 3, 3));
	        digraph.addEdge(new DirectedEdge(6, 4, 6));
	        digraph.addEdge(new DirectedEdge(6, 7, 2));
	        digraph.addEdge(new DirectedEdge(6, 8, 7));
	        digraph.addEdge(new DirectedEdge(7, 4, 9));
	        digraph.addEdge(new DirectedEdge(7, 5, 5));
	        digraph.addEdge(new DirectedEdge(7, 6, 2));
	        digraph.addEdge(new DirectedEdge(7, 8, 4));
	        digraph.addEdge(new DirectedEdge(8, 6, 7));
	        digraph.addEdge(new DirectedEdge(8, 7, 4));
	        Dijkstra dijkstra = new Dijkstra(digraph, 2);
	        for (DirectedEdge edge : dijkstra.edgeTo) {
                // 输出结果
	            System.out.println(edge);
	        }
	    }
	}

结果是：

	1->0  1.00
	0->1  1.00
	1->2  3.00
	4->3  2.00
	2->4  1.00
	4->5  3.00
	3->6  3.00
	6->7  2.00
	7->8  4.00

基于邻接矩阵的Dijkstra算法，它求解的是s到任意一点的最短路径，时间复杂度是O(n<sup>2</sup>). 如果想要求解任意一点到其他点的最短路径，那么时间复杂度是O(n<sup>3</sup>)：

    public Dijkstra(int s, double[][] G, double[] D, int[] P) {
        int v,w,k = 0;
        double min;
        boolean[] visited = new boolean[G.length];
        for (v=0;v<G.length;v++) {
            D[v] = G[s][v];
            P[v] = 0;
        }
        D[s] = 0;
        visited[s] = true;
        for (v=1;v<G.length;v++) {
            min = Double.POSITIVE_INFINITY;
            for (w =0;w<G.length;w++) {
                if (!visited[w]&&D[w]<min) {
                    k = w;
                    min = D[w];
                }
            }
            visited[k] = true;
            for (w=0;w<G.length;w++) {
                if (!visited[w]&&(min+G[k][w]<D[w])) {
                    D[w] = min + G[k][w];
                    P[w] = k;
                }
            }
        }
    }

    public static void main(String ...args) {
        double inf = Double.POSITIVE_INFINITY;
        double[][] G = new double[][]{
                {0,  1,  5,  inf,inf,inf,inf,inf,inf},
                {1,  0,  3,  7,  5,  inf,inf,inf,inf},
                {5,  3,  0,  inf,1,  7,  inf,inf,inf},
                {inf,7,  inf,0,  2,  inf,3,  inf,inf},
                {inf,5,  1,  2,  0,  3,  6,  9,  inf},
                {inf,inf,7,  inf,3,  0,  inf,5,  inf},
                {inf,inf,inf,3,  6,  inf,0,  2,  7},
                {inf,inf,inf,inf,9,  5,  2,  0,  4},
                {inf,inf,inf,inf,inf,inf,7,  4,  0}
        };
        int[] P = new int[G.length];
        double[] D = new double[G.length];
        Dijkstra dijkstra1 = new Dijkstra(0, G, D, P);
        for (int p : P) {
            System.out.print(p + " ");
        }
        System.out.println();
        for (double d : D) {
            System.out.print(d + " ");
        }
    }

输出结果：

	0 0 1 4 2 4 3 6 7 
	0.0 1.0 4.0 7.0 5.0 8.0 10.0 12.0 16.0 

#### 5.2.3 Floyd算法

Floyd算法可以求解出任意一点到其他顶点的最短路径，它的时间复杂度也是O(n<sup>3</sup>)：

    public static void Shortest_Floyd(int[][] P, double[][] D) {
        int v,w,k;
        
        for (v = 0;v<D.length;v++) {
            for (w = 0;w < D.length; w++) {
                P[v][w] = w;
            }
        }
        
        for (k=0;k<D.length;k++) {
            for (v=0;v<D.length;v++) {
                for (w=0;w<D.length;w++) {
                    if (D[v][w]>D[v][k]+D[k][w]) {
                        D[v][w] = D[v][k] + D[k][w];
                        P[v][w] = P[v][k];
                    }
                }
            }
        }
    }
    
    public static void main(String  ...args) {
        double inf = Double.POSITIVE_INFINITY;
        double[][] G = new double[][]{
            {0,  1,  5,  inf,inf,inf,inf,inf,inf},
            {1,  0,  3,  7,  5,  inf,inf,inf,inf},
            {5,  3,  0,  inf,1,  7,  inf,inf,inf},
            {inf,7,  inf,0,  2,  inf,3,  inf,inf},
            {inf,5,  1,  2,  0,  3,  6,  9,  inf},
            {inf,inf,7,  inf,3,  0,  inf,5,  inf},
            {inf,inf,inf,3,  6,  inf,0,  2,  7},
            {inf,inf,inf,inf,9,  5,  2,  0,  4},
            {inf,inf,inf,inf,inf,inf,7,  4,  0}
        };
        int[][] P = new int[9][9];
        Shortest_Floyd(P, G);
        for (int[] pp : P) {
            for (int p : pp) {
                System.out.print(p + " ");
            }
            System.out.println();
        }
        for (double[] gg : G) {
            for (double g : gg) {
                System.out.print(g + " ");
            }
            System.out.println();
        }
    }

结果是：

	0 1 1 1 1 1 1 1 1 
	0 1 2 2 2 2 2 2 2 
	1 1 2 4 4 4 4 4 4 
	4 4 4 3 4 4 6 6 6 
	2 2 2 3 4 5 3 3 3 
	4 4 4 4 4 5 7 7 7 
	3 3 3 3 3 7 6 7 7 
	6 6 6 6 6 5 6 7 8 
	7 7 7 7 7 7 7 7 8 
	
	0.0 1.0 4.0 7.0 5.0 8.0 10.0 12.0 16.0 
	1.0 0.0 3.0 6.0 4.0 7.0 9.0 11.0 15.0 
	4.0 3.0 0.0 3.0 1.0 4.0 6.0 8.0 12.0 
	7.0 6.0 3.0 0.0 2.0 5.0 3.0 5.0 9.0 
	5.0 4.0 1.0 2.0 0.0 3.0 5.0 7.0 11.0 
	8.0 7.0 4.0 5.0 3.0 0.0 7.0 5.0 9.0 
	10.0 9.0 6.0 3.0 5.0 7.0 0.0 2.0 6.0 
	12.0 11.0 8.0 5.0 7.0 5.0 2.0 0.0 4.0 
	16.0 15.0 12.0 9.0 11.0 9.0 6.0 4.0 0.0 


