# 1、无向图

## 1.1 图的数据结构定义

在[基本数据结构](https://github.com/Shouheng88/Java-Programming/blob/master/Data%20Structure/1.%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.md)中已经提及过图的表示的数据结构，这里我们使用链接表的方式来存储图并对它的内部的方法进行一些拓展。下面是一份完整的图的定义：[Graph.java](http://algs4.cs.princeton.edu/41graph/Graph.java.html "Graph.java")

	public class Graph {
	    private final int V; // 顶点数目
	    private int E; // 边的数目
	    private Bag<Integer>[] adj; // 邻接表
	
	    public Graph(int V) {
	        this.V = V;
	        adj = new Bag[V];
	        for (int i=0;i<V;i++) {
	            adj[i] = new Bag<Integer>();
	        }
	    }
	
	    public int V() { return V; }
	
	    public int E() { return E; }
	
	    /**
	     * 向无向图中添加一条边vw */
	    public void addEdge(int v, int w) {
	        adj[w].add(v);
	        adj[v].add(w);
	        E++;
	    }
	
	    public Iterable<Integer> adj(int v) { return adj[v]; }
	
	    private void validateVertex(int v) {
	        if (v < 0 || v >= V) {
	            throw new IllegalArgumentException("vertex " + v + " is not between 0 and " + (V-1));
	        }
	    }
	
	    @Override
	    public String toString() {
	        StringBuilder sb = new StringBuilder();
	        for (int i=0;i<V;i++) {
	            sb.append(i);
	            sb.append(" : ");
	            for (Integer w : adj[i]) {
	                sb.append(w);
	                sb.append(" ");
	            }
	            sb.append("\n");
	        }
	        return sb.toString();
	    }
	}

## 1.2 图的常用处理代码

下面的代码是在实际应用中经常使用到的图的一些处理逻辑，稍加思考就可以实现这些方法：

    /**
     * 计算图graph中的顶点v的度数 */
    public static int degree(Graph graph, int v) {
        int degree = 0;
        for (Integer w : graph.adj(v)) {
            degree++;
        }
        return degree;
    }

    /**
     * 计算图graph中最大度数 */
    public static int maxDegree(Graph graph) {
        int max = 0;
        for (int i=0;i<graph.V();i++) {
            int degree = degree(graph, i);
            max = degree > max ? degree : max;
        }
        return max;
    }

    /**
     * 计算所有顶点的平均度数 */
    public static double aveDegree(Graph graph) {
        // 图的度数的之和 = 2 * 图的边的数目
        return 2.0 * graph.E() / graph.V();
    }

    /**
     * 计算图中的自环的数目 */
    public static int numSelfLoops(Graph graph) {
        int count = 0;
        for (int i=0;i<graph.V();i++) { // 遍历所有结点i
            for (Integer w : graph.adj(i)) { // 结点i关联的结点w中是否包含i
                if (i == w) {
                    count++;
                }
            }
        }
    //  return count; // 错误！每个边都被记过两次
        return count / 2;
    }
