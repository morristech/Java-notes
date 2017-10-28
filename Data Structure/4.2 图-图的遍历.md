# 2、图的遍历

## 2.1 深度优先搜索算法

所谓深度优先就是当对图进行遍历时，每次遇到了分支，就选择某个分支走下去，当把一个分支走到底之后，再回到分支的地方，沿另一个分支走下去，直到遍历完所有结点。不过在每次访问完毕一个结点之后就会标记该结点为已访问，所以相同的结点不会被访问两次。因为每次都走到底，所以叫做深度优先搜索算法。

下面是深度优先搜索算法的方法框架：

    private boolean[] marked;
    private void dfs(Graph graph, int v) {
        marked[v] = true;
        for (int w : graph.adj(v)) {
            if (!marked[w]) {
                // ...逻辑
                dfs(graph, w);
            }
        }
    }

以上就是基本的深度优先搜索算法了，可以将一些其他的逻辑添加到if语句内部来实现想要的功能。可见在每次遇到结点v的时候，都会从v的所有子结点中选择一个（使用for循环来实现），这样就实现了“深度优先”的目的。

### 2.1.1 使用深度优先搜索算法寻找路径

下面是使用深度优先搜索算法来查找以source为原点，到任意顶点v的算法。这里只是在深度优先搜索算法的框架基础之上增加了`edgeTo[w] = v;`一行代码用于标记走过的路径。另外，这里还给出了使用栈来存储路径的方法`pathTo(int v)`。

    private boolean[] marked;
    private int[] edgeTo;
    private int count;
    private final int source;

    public DeepFirstSearch(Graph graph, int source) {
        marked = new boolean[graph.V()];
        edgeTo = new int[graph.V()];
        this.source = source;
        dfs(graph, source);
    }

    private void dfs(Graph graph, int v) {
        marked[v] = true;
        count++;
        for (int w : graph.adj(v)) {
            if (!marked[w]) {
                edgeTo[w] = v;
                dfs(graph, w);
            }
        }
    }

    public boolean hasPathTo(int v) {
        return marked[v];
    }

    public Iterable<Integer> pathTo(int v) {
        if (!hasPathTo(v)) {
            return null;
        }
        Stack<Integer> path = new Stack<Integer>();
        for (int i = v; i != source; i = edgeTo[i]) {
            path.push(i);
        }
        path.push(source);
        return path;
    }

## 2.2 广度优先搜索算法

与深度优先搜索算法不同，广度优先搜索算法可以得到从指定的结点到某个结点的最短路径。下面是广度优先搜索算法：

    private void bfs(Graph graph, int source) {
        marked[source] = true;
        Queue<Integer> queue = new Queue<Integer>();
        queue.enqueue(source);
        while (!queue.isEmpty()) {
            int v = queue.dequeue();
            for (Integer w : graph.adj(v)) {
                if (!marked[w]) {
                    marked[w] = true;
                    // ...用户逻辑
                    queue.enqueue(w);
                }
            }
        }
    }

可见，在广度优先搜索算法中兵没有使用递归来实现，而是使用了一个队列，每次访问结点的时候都会先弹出一个结点，然后对该结点的子结点进行遍历并将没有被访问过的结点放进队列中。

### 2.2.1 使用广度优先搜索算法寻找路径

将上面的广度优先搜索算法中的注释掉的“用户逻辑”替换成`edgeTo[w] = v;`一样可以实现查询从某个结点到指定结点的路径的目的。

使用广度优先搜索算法得到的结果和使用深度优先搜索算法的区别就在于，使用深度优先搜索的时候获得得都是从某个结点到指定结点的最短路径。

## 2.3 更多应用

### 2.3.1 使用深度优先和广度优先搜索算法遍历当前目录下的所有目录和文件

下面是使用深度优先和广度优先遍历当前目录下面的所有子目录和文件的示例。注意：

1. 在广度优先搜索算法中，我们使用了LinkedList来实现队列的功能；
2. 不能在使用foreach循环遍历容器的时候修改容器中的内容

程序：

    public static void main(String ...args) {
        File root = new File(".");
        bfs(root); // 广度优先遍历
        dfs(root); // 深度优先遍历
    }

    private static void bfs(File root) {
        Queue<File> files = new LinkedList<File>();
        File[] listFiles = root.listFiles();
        if (listFiles != null) {
            files.addAll(Arrays.asList(listFiles));
        }
        while (!files.isEmpty()) {
            int size = files.size();
            for (int i=0;i<size;i++) {
                File file = files.poll();
                if (file.isDirectory()) {
                    File[] lf = file.listFiles();
                    if (lf != null) {
                        files.addAll(Arrays.asList(lf));
                    }
                }
                System.out.println(file.getName());
            }
        }
    }

    private static void dfs(File file) {
        if (file.isDirectory()) {
            File[] files = file.listFiles();
            if (files != null) {
                for (File f : files) {
                    dfs(f);
                }
            }
        }
        System.out.println(file.getName());
    }