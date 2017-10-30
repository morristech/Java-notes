## 迭代器模式

迭代器模式提供了一种方法顺序访问一个聚合对象的各个元素，而又不暴露其内部的表示。

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1506060512778&di=7e38debb7b37cd54cfb305d6f8dba21f&imgtype=0&src=http%3A%2F%2Fimages.cnitblog.com%2Fblog%2F507671%2F201304%2F30141826-36e146674f974c7b9a674d0891630ddd.jpg)

以上是迭代器模式的类图。它包含了四种角色：

1. **抽象容器(Aggregate)**：一般是一个接口，提供一个iterator()方法，例如java中的Collection接口，List接口，Set接口等；
2. **具体容器(ConcreteAggregate)**：容器就是数据的集合，它用来保存一系列的数据；
3. **抽象迭代器(Iterator)**：定义遍历元素所需要的方法，一般来说会有这么三个方法：取得第一个元素的方法first()，取得下一个元素的方法next()，判断是否遍历结束的方法isDone()（或者叫hasNext()），移出当前对象的方法remove()；
4. **具体迭代器(ConcreteIterator)**：实现迭代器接口中定义的方法，完成集合的迭代。

如果是在Java中的话，我们一般通过为一个容器实现`Iterable`接口来实现迭代器，该接口包含一个`iterator()`方法，用来返回一个迭代器实例（具体迭代器）。这个迭代器实例需要实现`Iterator`（迭代器）接口。

下面是Java中迭代器的一个使用的示例:

	public class Queue<E> implements Iterable<E>{
	    private Node<E> first, last;
	    private int size;
	
	    @Override
	    public Iterator<E> iterator() {
	        return new Iterator<E>() {
	
	            Node<E> current = first;
	
	            @Override
	            public boolean hasNext() {
	                return current != null;
	            }
	
	            @Override
	            public E next() {
	                E element = current.element;
	                current = current.next;
	                return element;
	            }
	
	            @Override
	            public void remove() {}
	        };
	    }
	
	    private static class Node<E> {
	        E element;
	        Node<E> next;
	
	        Node(E element, Node<E> next) {
	            this.element = element;
	            this.next = next;
	        }
	    }
	
	    public void enqueue(E element) {
	        size++;
	        Node<E> node = new Node<E>(element, null);
	        if (first == null) {
	            first = node;
	            last = node;
	            return;
	        }
	        last.next = node;
	        last = node;
	    }
	
	    public E dequeue() {
	        if (size == 0) {
	            throw new UnsupportedOperationException("The queue is empty.");
	        }
	        size--;
	        E element = first.element;
	        first = first.next;
	        return element;
	    }
	
	    public boolean isEmpty() {
	        return size == 0;
	    }
	}

以上是一份队列的实现，这里我们将该Queue类实现了Iterable接口，并在iterator()方法中遍历队列中的元素。

## 组合模式

将对象组合成树形结构以表示“部分整体”的层次结构。组合模式使得用户对单个对象和使用具有一致性。

![](http://my.csdn.net/uploads/201206/26/1340694955_4501.jpg)

以上是组合模式的类图，它让我们可以像树形结构一样来对容器中的元素进行遍历。从上面的图中可以看出，它提出了两种的概念来抽象组合模式中的元素：一个是没有任何子节点的Leaf结点，一个是包含子节点的Composite结点。

组合模式应用场景不难理解，比如文件系统中，有时候文件夹下面包含文件和文件，被包含的文件夹中可能又包含文件夹和文件。组合模式适用于这种具有层次结构的模型。

下面是组合模式的一种使用示例：

Component类定义了组合模式的组件需要的依稀方法：

	public abstract class Component <E> {
	
	    protected abstract void operation();
	
	    protected abstract void add(Component<E> component);
	
	    protected abstract void remove(Component<E> component);
	
	    protected abstract Component<E> getChild(int index);
	}

叶子结点，不具有子结点的类型：

	public class Leaf <E> extends Component<E>{
	    private E e;
	
	    public Leaf(E e) {
	        this.e = e;
	    }
	
	    @Override
	    protected void operation() {
	        System.out.print(e.toString() + " ");
	    }
	
	    @Override
	    protected void add(Component<E> component) {}
	
	    @Override
	    protected void remove(Component<E> component) {}
	
	    @Override
	    protected Component<E> getChild(int index) {
	        return null;
	    }
	}

Composite，包含了一个组件列表，可以为其添加子Composite或者叶子结点：

	public class Composite<E> extends Component<E> implements Iterable<Component<E>>{
	    private List<Component<E>> children = new ArrayList<Component<E>>();
	
	    @Override
	    protected void operation() {
	        for (Component component : children) {
	            component.operation();
	        }
	    }
	
	    @Override
	    protected void add(Component<E> component) {
	        children.add(component);
	    }
	
	    @Override
	    protected void remove(Component<E> component) {
	        component.remove(component);
	    }
	
	    @Override
	    protected Component<E> getChild(int index) {
	        return children.get(index);
	    }
	
	    public Iterator<Component<E>> createIterator() {
	        return children.iterator();
	    }
	
	    @Override
	    public Iterator<Component<E>> iterator() {
	        return new Iterator<Component<E>>() {
	
	            Stack<Iterator<Component<E>>> stack = new Stack<Iterator<Component<E>>>();
	            {
	                stack.push(children.iterator());
	            }
	
	            @Override
	            public boolean hasNext() {
	                if (stack.isEmpty()) {
	                    return false;
	                } else {
	                    Iterator<Component<E>> iterator = stack.peek();
	                    if (!iterator.hasNext()) {
	                        stack.pop();
	                        return hasNext();
	                    } else {
	                        return true;
	                    }
	                }
	            }
	
	            @Override
	            public Component<E> next() {
	                Iterator<Component<E>> iterator = stack.peek();
	                Component<E> component = iterator.next();
	                if (component instanceof Composite) {
	                    stack.push(((Composite<E>) component).createIterator());
	                }
	                return component;
	            }
	
	            @Override
	            public void remove() {}
	        };
	    }
	}

在上面的Composite类中，我们让该类实现了Iterable接口，这样它就可以使用for循环来进行迭代了。我们可以将其称为组合迭代器，注意一下它的实现方式。

组合对象将组合对象和对象一视同仁，它提供了一个结构，可以同时包含个别对象和组合对象。