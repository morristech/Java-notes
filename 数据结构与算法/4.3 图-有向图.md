# 图

## 3、有向图

### 3.1 有向图的数据结构

与无向图的不同之处在于添加一条边的时候的逻辑，另外在有限图中增加了一个将所有的边取反的`reverse()`方法。

下面的代码是通过继承上例中的`Graph.class`实现的无向图`Digraph.class`([Digraph.java](http://algs4.cs.princeton.edu/42digraph/Digraph.java.html "Digraph.java"))：

	public class Digraph extends Graph{
	
	    public Digraph(int V) {
	        super(V);
	    }
	
	    public Digraph(In in) {
	        super(in);
	    }
	
	    @Override
	    public void addEdge(int v, int w) {
	        adj[v].add(w);
	        E++;
	    }
	
	    public Digraph reverse() {
	        Digraph digraph = new Digraph(V);
	        for (int i=0;i<V;i++) {
	            for (int v : adj[i]) {
	                digraph.addEdge(v, i);
	            }
	        }
	        return digraph;
	    }
	}

有向图在实际应用中非常具有价值，以下是它的一些常见的应用：

1. 标记-清除的垃圾收集机制：借助有向图来表达各个对象之间的依赖关系；
2. 调度问题：同样借助有向图来表达各个任务之间的依赖关系；

### 3.2 有向图寻找路径

有向图和无向图的深度优先搜索算法和广度优先搜索算法结构完全相同，这里不再给出。

### 3.3 有向图中的环检测

在许多实际问题中是不允许出现有向环的，因此需要提前对有向图中的环进行检测。有了深度优先搜索算法要实现有向环检测并不难。之前我们在寻找路径的时候使用以某个顶点为源点的方式进行深度优先搜索，在搜素有向环的时候我们需要遍历图中的所有顶点，并依次按照将各顶点作为源点的方式进行深度优先搜索。只是这里我们使用一个布尔类型的数组用于标记某个顶点是否在环上，如果两次访问同一顶点就表示存在有向环。

    private boolean[] marked;
    private int[] edgeTo;
    private Stack<Integer> cycle; // 有向环中所有顶点
    private boolean[] onStack; // 递归调用的栈上的所有顶点

    public DirectedCycle(Digraph digraph) {
        onStack = new boolean[digraph.V()];
        edgeTo = new int[digraph.V()];
        marked = new boolean[digraph.V()];
        for (int v=0;v<digraph.V();v++) {
            if (!marked[v]) {
                dfs(digraph, v);
            }
        }
    }

    private void dfs(Digraph digraph, int v) {
        marked[v] = true;
        onStack[v] = true; // 在栈上记录当前顶点
        for (int w : digraph.adj(v)) {
            if (hasCycle()) { // 找到一个环即退出
                return;
            } else if (!marked[w]) { // 深度优先搜索算法
              edgeTo[w] = v;
              dfs(digraph, w);
            } else if (onStack[w]) { // 当找到了环的时候记录该换的记录信息
                cycle = new Stack<Integer>();
                for (int x=v;x!=w;x=edgeTo[x]) {
                    cycle.push(x);
                }
                cycle.push(w);
                cycle.push(v);
            }
        }
        onStack[v] = false; // 递归结束的时候将所有的记录复原，以便于下次使用
    }

    public boolean hasCycle() {
        return cycle != null;
    }

    public Iterable<Integer> cycle() {
        return cycle;
    }

### 3.4 有向无环图的拓扑排序

当且仅当一幅有向图是无环图的时候才能进行拓扑排序。拓扑排序也是借助于深度优先搜索算法来实现。

一幅有向无环图的拓扑顺序就是所有顶点的逆后序排列。就是说如果你在处理任务调度问题的时候，想要得到一个正确的调度方案就需要使用深度优先算法来获取图的逆后序。

常用的三种排序方式：

1. **前序**：在递归调用**之前**将顶点加入**队列**；
2. **后序**：在递归调用**之后**将顶点加入**队列**；
3. **逆后序**：在递归调用**之后**将顶点压入**栈**。

下面就是一份使用Java实现的拓扑排序：

	public class DepthFirstOrder {
	    private boolean[] marked;
	    private Queue<Integer> pre;
	    private Queue<Integer> post;
	    private Stack<Integer> revPost;
	
	    public DepthFirstOrder(Digraph digraph) {
	        marked = new boolean[digraph.V()];
	        pre = new Queue<Integer>();
	        post = new Queue<Integer>();
	        revPost = new Stack<Integer>();
	        for (int v=0;v<digraph.V();v++) {
	            if(!marked[v]) {
                    dfs(digraph, v);
                }
	        }
	    }
	
	    private void dfs(Digraph digraph, int v) {
	        marked[v] = true;
	        pre.enqueue(v);
	        for (int w : digraph.adj(v)) {
	            if (!marked[w]) {
	                dfs(digraph, w);
	            }
	        }
	        post.enqueue(v);
	        revPost.push(v);
	    }
	
	    // getter pre post revPost
	}

### 3.5 有向图中的强连通分量

计算有向图中的强连通分量需要分成下面三步：

1. 给定一幅有向图，计算它的**反向图**的**逆后序**排列；
2. 在图G中使用上述计算得到的排序访问未被访问过的顶点；
3. 使用深度优先搜索，在同一个递归中被访问到的顶点都在同一个强连通分量中。

下面是获取图的强连通分量的算法

	private boolean[] marked;
    private int[] id;             // id[v] = id of strong component containing v
    private int count;            // number of strongly-connected components

    public KosarajuSharirSCC(Digraph G) {
        DepthFirstOrder dfs = new DepthFirstOrder(G.reverse());
        marked = new boolean[G.V()];
        id = new int[G.V()];
        for (int v : dfs.revPost()) {
            if (!marked[v]) {
                dfs(G, v);
                count++;
            }
        }
    }

	private void dfs(Digraph G, int v) {
        marked[v] = true;
        id[v] = count;
        for (int w : G.adj(v)) {
            if (!marked[w]) dfs(G, w);
        }
    }