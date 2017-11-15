# 图

## 4、最小生成树

最小生成树(MST)：给定一幅图是**加权无向图**，它的最小生成树是权值最小的生成树。

树的两个最重要的**性质**：

1. 用一条边连接树中的任意两个顶点都会产生一个新的环；
2. 从树中删除一条边将会得到两棵独立的树。

图的一种切分是将图的所有顶点分为两个非空且不重叠的集合，**横切边**是连接两个属于不同集合的顶点的边。

**切分定理**：在一幅加权图中，给定任意的切分，它的**横切边**中的权重最小者必然属于图的最小生成树。

上面的切分定理很容易证明，了解它的思想也就更容易理解最小生成树的算法实现原理。假如最小生成树为T，自小的切分边为e，现在假设e不在T中，而是f。如果我们在T中加入了e，那么T就构成了一个环，这时候如果我们将f删掉，那么e就和T剩下的部分构成了最小生成树，从而和开始为最小生成树矛盾。

### 4.1 加权无向图的数据类型

在加权无向图中，我们除了要定义一条边的两个顶点，还要定义边的权重。下面是加权无向图的数据结构，我们将边抽象成一个类`Edge.class`。

	public class Edge implements Comparable<Edge> { 
	    private final int v;
	    private final int w;
	    private final double weight;

	    public Edge(int v, int w, double weight) {
	        this.v = v;
	        this.w = w;
	        this.weight = weight;
	    }
	
	    public double weight() { return weight; }
	
        // 返回边的两个顶点中的任意一个，因为边是无向的
	    public int either() { return v; }
	
        // 在已知一个顶点vertex的情况下，获取边的另一个顶点
	    public int other(int vertex) {
	        if (vertex == v) return w;
	        else if (vertex == w) return v;
	        else throw new IllegalArgumentException("Illegal endpoint");
	    }

	    @Override
	    public int compareTo(Edge that) {
	        return Double.compare(this.weight, that.weight);
	    }
	}

有了边的类之后，我们使用它来定义图的数据类型。这里adj是一个数组，数组的索引是顶点的值，数组中的背包中存储的是该顶点相连的所有的边。

	public class EdgeWeightedGraph {
	    private final int V; // 顶点总数
	    private int E; // 边的总数
	    private Bag<Edge>[] adj; // 邻接表
	    
	    public EdgeWeightedGraph(int V) {
	        this.V = V;
	        this.E = 0;
	        adj = (Bag<Edge>[]) new Bag[V];
	        for (int v = 0; v < V; v++) {
	            adj[v] = new Bag<Edge>();
	        }
	    }
	
	    public int V() { return V; }
	
	    public int E() { return E; }
	
        // 无向图中加入新的边的时候，要在两个顶点的边的集合中都加入该边的记录
	    public void addEdge(Edge e) {
	        int v = e.either();
	        int w = e.other(v);
	        adj[v].add(e);
	        adj[w].add(e);
	        E++;
	    }
	
	    public Iterable<Edge> adj(int v) { return adj[v]; }

	    public Iterable<Edge> edges() {
	        Bag<Edge> list = new Bag<Edge>();
	        for (int v = 0; v < V; v++) {
	            int selfLoops = 0;
	            for (Edge e : adj(v)) {
	                if (e.other(v) > v) {
	                    list.add(e);
	                } else if (e.other(v) == v) {
	                    if (selfLoops % 2 == 0) list.add(e);
	                    selfLoops++;
	                }
	            }
	        }
	        return list;
	    }
	}

### 4.2 Prim算法

其实要理解Prim算法很简单，它就是从一个结点开始，将该与该结点相连的边作为最小生成树的一条边，然后再从与该边的另一个顶点相连的所有边中选择一个权重最小的边，依次进行下去。就是说，每次都找该与该顶点相连的所有边中权重最小的一个。只是要注意的事情是，在选择边的时候已经加入了边不能重复加入，还不能N引入环。

至于为什么每次都要选择与一个顶点相连的最小的边：和上面讨论切分定理类似，假如我们认定它不在树中，那么如果我们将其加入树中就想成了环，然后将另一条边去掉就得到了一个权重更小的生成树。显然与假设是矛盾的。

#### 4.2.1 Prim算法的延迟实现

![](http://chuantu.biz/t6/111/1508919179x1942832654.png)

我们可以用上面的图来说明Prim算法的基本思路。假如我们从点v0开始，那么：

1. 首相将v0的全部邻接边放进队列中，然后我们从队列中找出一个最小的边作为最小生成树的边，即v0->v1，并从队列中删除；
2. 我们将上述得到的最小生成树的边v0->v1中的v1的全部邻接边，去掉与v0相连的边即v1->v0（因为它已经被添加了），全部放入到队列中，并将v1标记为“已访问”；
3. 然后我们再从队列的所有边中找出最小的一个来将其作为最小生成树的边，并将其删掉，即v1->v2，然后将v2的所有邻接边，去掉v2-v1（已经访问）和v2->v0（会生成环），放入到队列中；
4. ……依次类推

所以，我们其实每次都是从队列中，也就是已经加入的顶点的邻接边列表中选出一个最小的边，来作为最小生成树的边。还要注意，我们每次要将指定顶点的邻接边中，另一个顶点已经被标记的边去掉。因为，这种边要么是已经添加了的边，要么会生成环。

下面是使用java实现的求解最小生成树的算法：

在这里，我们将结果，即最小生成树的各个边，放进result中。这里的getShortestEdge()方法可以进行优化，它每次需要遍历队列来获取一个最小的边。这是没有必要的，我们可以使用一个优先队列，来更容易地获取到最小的边。

还要注意，如果一个结点被标记为已访问，那么我们需要将队列中到达它的边都移除掉。比如将2标记为已经访问之后，虽然我们没有将2->0添加到队列中，但是队列中的0->2却仍然存在。因此，如果我们将2标记为已访问，那么我们需要将0->2从队列中移除（也就是队列中边的达到的顶点是2所有的边）。

为什么将指向2的所有的边移除呢？是这样，因为我们在对无向图进行存储的时候实际上存储了两份，也就是对结点0有0->2，对结点2有2->0. 当我们将结点0的邻接边放进队列中时，实际上是放进了从0指出的边。对其他的每个顶点也是如此，即我们只将指出的边加入队列。而当有两个不同的边指向同一顶点的时候，肯定要生成环了，即1->2和0->2必定生成环。所以，既然我们已经将指定的顶点加入到了“已访问”中，那么我们就应该将指向它的其他的边从队列中移除出去，因为树中已经有一条指向它的边了。

    private static List<DirectedEdge> result = new LinkedList<DirectedEdge>();

    public static void PrimeTree(EdgeWeightedDigraph digraph, int s) {
        List<DirectedEdge> edges = new LinkedList<DirectedEdge>();
        boolean[] marked = new boolean[digraph.V()];
        for (DirectedEdge edge : digraph.adj(s)) {
            edges.add(edge);
        }
        marked[s] = true;
        while (!edges.isEmpty()) {
            DirectedEdge shortest = getShortestEdge(edges, marked);
            edges.remove(shortest);
            result.add(shortest);
            marked[shortest.to()] = true;
            for (DirectedEdge edge : digraph.adj(shortest.to())) {
                if (!marked[edge.to()]) {
                    edges.add(edge);
                }
            }
            rmMarked(edges, marked);
        }
    }

    private static void rmMarked(List<DirectedEdge> edges, boolean[] marked) {
        for (int i=0;i<edges.size();i++) {
            DirectedEdge edge = edges.get(i);
            if (marked[edge.to()]) {
                i--; // 删除一个之后，索引减1
                edges.remove(edge);
            }
        }
    }

    private static DirectedEdge getShortestEdge(List<DirectedEdge> edges) {
        double min = Double.POSITIVE_INFINITY;
        DirectedEdge minEdge = null;
        for (DirectedEdge edge : edges) {
            if (edge.weight() < min) {
                min = edge.weight();
                minEdge = edge;
            }
        }
        return minEdge;
    }

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
        PrimeTree(digraph, 0);
        for (DirectedEdge edge : result) {
            System.out.println(edge);
        }
    }

输出结果：

	0->1  1.00
	1->2  3.00
	2->4  1.00
	4->3  2.00
	4->5  3.00
	3->6  3.00
	6->7  2.00
	7->8  4.00

**性能**

Prim算法的延时版本中实现一幅含有V个顶点和E条边的连通加权无向图的最小生成树所需的空间与E成正比，所需的时间与ElogE成正比。

#### 4.2.2 Prim算法的即时实现

下面是Prim算法的即时实现，它依赖于[IndexMinPQ.java](http://algs4.cs.princeton.edu/43mst/IndexMinPQ.java.html)。

	public class PrimMST {
	    private Edge[] edgeTo;        // edgeTo[v] = shortest edge from tree vertex to non-tree vertex
	    private double[] distTo;      // distTo[v]=weight of shortest such edge
	    private boolean[] marked;     // 如果顶点v在树中，那么marked[v]=true
	    private IndexMinPQ<Double> pq; // 有效的横切边
	
	    public PrimMST(EdgeWeightedGraph G) {
	        edgeTo = new Edge[G.V()];
	        distTo = new double[G.V()];
	        marked = new boolean[G.V()];
	        pq = new IndexMinPQ<Double>(G.V());
	        for (int v = 0; v < G.V(); v++) // 使用最大的Double类型初始化distTo
	            distTo[v] = Double.POSITIVE_INFINITY;
	        for (int v = 0; v < G.V(); v++)
	            if (!marked[v]) prim(G, v);
	    }
	
	    private void prim(EdgeWeightedGraph G, int s) {
	        distTo[s] = 0.0; // 用顶点0和权重0.0初始化pq
	        pq.insert(s, distTo[s]);
	        while (!pq.isEmpty()) {
	            int v = pq.delMin();
	            scan(G, v);
	        }
	    }
	
	    private void scan(EdgeWeightedGraph G, int v) {
	        marked[v] = true;
	        for (Edge e : G.adj(v)) {
	            int w = e.other(v);
	            if (marked[w]) continue; // v-w is 失效的边
	            if (e.weight() < distTo[w]) { // 实际上是比较操作来得到与顶点v相连的边种权重最小一个
	                distTo[w] = e.weight();
	                edgeTo[w] = e;
	                if (pq.contains(w)) pq.decreaseKey(w, distTo[w]);
	                else                pq.insert(w, distTo[w]);
	            }
	        }
	    }
	}

**性能**

Prim算法的即使实现计算一个含有V个顶点和E条边的连通加权无向图的最小生成树所需的空间和V成正比，所需的时间和ElogV成正比。

### 4.3 Kruskal算法

Kruskal算法中大部分内容都不难理解，它的基本思路是：每次从最小的边中选择一条，只要避免加入新的边之后生成了环就可以了。在下面的算法中，我们使用parent数组来避免生成环。这里，我们先对图的所有的边进行选择，我们只选择出了起点的编号小于终点的编号的边，然后，我们使用了快速排序算法对边按照从小到大的顺序进行排序。

这里关键是parent数组如何理解，对上面的图我们可以这样来理解：假如我们先加入了边0-1，这时候parent[0]=1，然后我们加入了边1-2，这时parent[1]=2. 此时，如果边0-2比较小，我们试图将其加入进来，就会发生以下现象：我们从0找到parent[0]=1，然后找到parent[1]=2，即找到了2，而对2使用find方法的时候直接回找到2. 此时2=2，因此就无法将边0-2添加到结果集中。

也就是说，在parent中保存的内容相当于一条链路，我们可以往下一直找到链路的终点。当新加入一条边的时候，我们也是对新加入的边，沿着这条链路来寻找，而且只要加入的点在这条链上面，无论你从哪个位置开始都能找到链的末端。

比如上面的例子中，如果我们在加入0-2之前，先加入了1-3，那么此时parent[2]=3. 这样就形成了这么一条链0-1-2-3，所以这要你要加入的边的两个顶点都在这条链上面，那么它们在使用find方法寻找的时候总能找到3，所以就加入不进去了。

    public static void KruskalTree(EdgeWeightedDigraph digraph) {
        // parent在这里的主要作用是避免添加边的时候生成环
        int[] parent = new int[digraph.V()];
        // 构建一个边的数组，包含不重复的边，数组的大小是边总数的一般,0->1和1->0一样
        DirectedEdge[] edges = new DirectedEdge[digraph.E()/2];
        int i = 0;
        // 将边添加到数组中
        for (int v=0;v<digraph.V();v++) {
            for (DirectedEdge edge : digraph.adj(v)) {
                // 我们只添加起点小于终点的边
                if (edge.from() < edge.to()) {
                    edges[i++] = edge;
                }
            }
        }

        // 按照边的权重的大小，使用快速排序
        Quick.sort(edges);

        // 对所有的边进行遍历
        int n,m;
        for (i=0;i<digraph.E()/2;i++) {
            DirectedEdge edge = edges[i];
            n = find(parent, edge.from());
            m = find(parent, edge.to());
            if (n != m) {
                parent[n] = m;
                System.out.println(edge);
            }
        }

        for (int p : parent) {
            System.out.print(p + " ");
        }
    }

    private static int find(int[] parent, int f) {
        while (parent[f] > 0) {
            f = parent[f];
        }
        return f;
    }

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
        KruskalTree(digraph, 0);
    }

输出结果：

	0->1  1.00
	2->4  1.00
	6->7  2.00
	3->4  2.00
	3->6  3.00
	4->5  3.00
	1->2  3.00
	7->8  4.00
	1 5 4 4 7 8 7 5 0 

Kruskal算法计算一个含有V个顶点和E条边的连通加权无向图的最小生成树所需的空间和E成正比，所需的时间和ElogE成正比。

Kruskal算法只要针对边来展开，边的数目比较少的时候，效率比较高；而Prime算法对于稠密图，即边的数目非常多的算法好一些。