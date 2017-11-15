# 乐观锁和悲观锁

### 1、悲观锁的实现

#### 1.1 悲观锁介绍（百科）：

悲观锁，正如其名，它指的是对数据被外界（包括本系统当前的其他事务，以及来自外部系统的事务处理）修改持保守态度，因此，在整个数据处理过程中，将数据处于锁定状态。悲观锁的实现，往往依靠数据库提供的锁机制（也只有数据库层提供的锁机制才能真正保证数据访问的排他性，否则，即使在本系统中实现了加锁机制，也无法保证外部系统不会修改数据）。

#### 1.2 使用场景举例：以MySQL InnoDB为例

商品goods表中有一个字段status，status为1代表商品未被下单，status为2代表商品已经被下单，那么我们对某个商品下单时必须确保该商品status为1。假设商品的id为1。
 
1.如果不采用锁，那么操作方法如下：

    // 1.查询出商品信息
    select status from t_goods where id=1;
    // 2.根据商品信息生成订单
    insert into t_orders (id,goods_id) values (null,1);
    // 3.修改商品status为2
    update t_goods set status=2;
 
上面这种场景在高并发访问的情况下很可能会出现问题。

前面已经提到，只有当goods status为1时才能对该商品下单，上面第一步操作中，查询出来的商品status为1。但是当我们执行第三步Update操作的时候，有可能出现其他人先一步对商品下单把goods status修改为2了，但是我们并不知道数据已经被修改了，这样就可能造成同一个商品被下单2次，使得数据不一致。所以说这种方式是不安全的。
 
2.使用悲观锁来实现：

在上面的场景中，商品信息从查询出来到修改，中间有一个处理订单的过程，使用悲观锁的原理就是，当我们在查询出goods信息后就把当前的数据锁定，直到我们修改完毕后再解锁。那么在这个过程中，因为goods被锁定了，就不会出现有第三者来对其进行修改了。

> 注：要使用悲观锁，我们必须关闭mysql数据库的自动提交属性，因为MySQL默认使用autocommit模式，也就是说，当你执行一个更新操作后，MySQL会立刻将结果进行提交。
 
我们可以使用命令设置MySQL为非autocommit模式：

    set autocommit=0;
 
设置完autocommit后，我们就可以执行我们的正常业务了。具体如下：

    // 0.开始事务
    begin;/begin work;/start transaction; (三者选一就可以)
    // 1.查询出商品信息
    select status from t_goods where id=1 for update;
    // 2.根据商品信息生成订单
    insert into t_orders (id,goods_id) values (null,1);
    // 3.修改商品status为2
    update t_goods set status=2;
    // 4.提交事务
    commit;/commit work;

注：上面的begin/commit为事务的开始和结束，因为在前一步我们关闭了mysql的autocommit，所以需要手动控制事务的提交，在这里就不细表了。
 
上面的第一步我们执行了一次查询操作：

    select status from t_goods where id=1 for update;

与普通查询不一样的是，我们使用了`select…for update`的方式，这样就通过数据库实现了悲观锁。此时在t_goods表中，id为1的 那条数据就被我们锁定了，其它的事务必须等本次事务提交之后才能执行。这样我们可以保证当前的数据不会被其它事务修改。

注：需要注意的是，在事务中，只有`SELECT ... FOR UPDATE`或`LOCK IN SHARE MODE`同一笔数据时会等待其它事务结束后才执行，一般`SELECT...`则不受此影响。拿上面的实例来说，当我执行`select status from t_goods where id=1 for update;`后。我在另外的事务中如果再次执行`select status from t_goods where id=1 for update;`则第二个事务会一直等待第一个事务的提交，此时第二个查询处于阻塞的状态，但是如果我是在第二个事务中执行`select status from t_goods where id=1;`则能正常查询出数据，不会受第一个事务的影响。

[来源：http://chenzhou123520.iteye.com/blog/1860954](http://chenzhou123520.iteye.com/blog/1860954)


