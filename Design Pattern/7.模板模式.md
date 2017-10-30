## 模板模式

模板模式是一种非常实用的设计模式，它可以为我们节省大量的重复代码。甚至可以说，如果不了解模板设计模式就没有充分利用面向对象的好处。所谓模板就是为业务逻辑提供一种模板框架，而至于这些模板的每个模块该如何实现，我们将其交给子类。也就是说，我们在顶层基类中定义一个业务逻辑模板，子类需要根据自己的需求实现该模板中的一些方法。

这里先给两个在实际应用中的例子，然后对模板模式的一些技巧进行总结。

### 第一种示例

现在我们有两个碎片(可以将碎片理解成一个页面)：`AssignmentFragment`和`NoteFragment`. 它们继承了一个共同顶层基类`BaseFragment`（我们使用这个类来定义一些子类中能用到的方法）. 

现在我们有一个定位的功能需求，在`AssignmentFragment`和`NoteFragment`中都需要这个功能。我们只希望实现一份定位的方法，然后在子类中调用，并将结果“返回”给子类。如果是在同一个类中获取位置，那么我们可以直接使用获取到的位置信息。但是现在在“父类”中进行定位，获取到的位置信息我们怎么将它们传递到“子类”中呢？对，使用“钩子”。我们只需要定义一个空的方法，将位置信息作为它的参数，然后如果我们在子类中需要位置信息，就可以覆写该方法，然后加入我们需要的逻辑。

下面是我们的一份实现代码实现：

	public abstract class BaseFragment {
	
	    protected void locate(){
	        // 使用异步方法进行定位，并将结果回调回来
	        LocationUtils.getInstance(getContext()).locate(new BDLocationListener() {
	            @Override
	            public void onReceiveLocation(BDLocation bdLocation) {
	                // 当获取到位置信息的时候调用这个方法
	                onGetLocation(bdLocation);
	            }
	        });
	    }
	
	    /**
	     * 子类在调用了tryToLocate()方法之后再调用本方法获取定位的结果并进行逻辑处理
	     *
	     * @param bdLocation 定位结果 */
	    protected void onGetLocation(BDLocation bdLocation) {}
	}

这是我们在基类中定义的关于获取位置的部分的代码。在这里我们使用异步的方式获取位置信息，当获取到位置信息的时候，我们将获取到的位置信息作为参数传递到onGetLocation()方法中。

在子类中，我们可以直接调用locate()方法来获取位置信息，然后选择覆写onGetLocation(BDLocation bdLocation)方法添加当获取到位置信息时候的逻辑。下面是它的实现：

	public class AssignmentFragment extends BaseFragment {

	    @Override
	    public void onClick(View view) {
	        // 调用父类中的方法进行定位
	        tryToLocate();
        }

	    @Override
	    protected void onGetLocation(BDLocation bdLocation) {
	        super.onGetLocation(bdLocation);
	        // 执行显示位置信息的逻辑
	    }
	}

这里我们在单击事件onClick(View view)方法中调用定位方法，然后当获取到位置信息的时候，“父类”（其实是当前类）会调用我们的onGetLocation(BDLocation bdLocation)方法，也就调用了我们的显示位置信息的逻辑。

其实这里所谓的定位方法在父类中的父类是当前类，也就是调用定位方法的类，因为继承了之后这个方法就属于自己了。而且我们在子类中选择实现onGetLocation(BDLocation bdLocation)方法并在其中添加显示位置的逻辑的效果也只是：当前类调用"自己"的获取位置的方法，当获取到位置之后，调用"自己"的显示位置的方法。

### 第二个示例

考虑我们正在做一个编辑界面，现在我们要为编辑界面的保存事件进行处理。我们将它的流程设计成这个样子

![](http://upload.ouliu.net/i/20171029162111xkrr6.png)

如图所示，在保存之前我们要校验输入信息的完整性，当信息完整的时候我们才进行保存，否则要给出指定的提示。然后，我们需要校验当前正在编辑的数据实体是新添加的实体还是对已经存在的实体进行修改。它们分别对应了插入和更新档SQL操作。当保存完毕的时候，我们需要给出保存成功的通知，并需要对当前页面的一些状态进行处理。

以上是相对简单的一种模板，如图所示，我们希望仅在顶层基类中设计这种模板，然后将一些个性化的实现交给子类来实现，以达到代码复用的目的。这里我们将一些需要子类实现的逻辑块标记成了绿色。

下面我们就对这种模板进行代码编写：

	public abstract class BaseModelFragment<T extends Model> extends BaseFragment

	    // 是否是新加入的数据实体的标记变量
	    private Boolean isNewModel;

	    // 获取数据实体对应的数据类型
	    protected abstract BaseStore getStoreOfModel();

	    // 获取数据实体
	    protected abstract T getModel();

	    // 校验输入信息的完整性
	    protected abstract boolean checkInputInfo();

	    // 保存编辑结果之前的处理
	    protected void beforeSaveOrUpdate(){};

		// 是否是新加入的数据实体
	    protected boolean isNewModel() {
	        if (isNewModel == null) {
	            isNewModel = getStoreOfModel().isNewModel(getModel().getCode());
	        }
	        return isNewModel;
	    }

	    // 保存数据实体
	    protected abstract void saveModel();

	    // 更新数据实体
	    protected abstract void updateModel();
	
	    // 保存之后的处理
	    protected void afterSaveOrUpdate() {}

		// 模板设计
	    protected boolean saveOrUpdateData() {
	        // 需要对输入的信息进行校验
	        if (!checkInputInfo()){
	            return false;
	        }
	
	        // 在持久化之前进行的操作
	        beforeSaveOrUpdate();
	
	        // 进行持久化操作
	        if (isNewModel()){
	            saveModel();
	        } else {
	            updateModel();
	        }
	
	        // 保存完毕之后该数据实体就不是最新的了
	        isNewModel = false;
	
	        // 弹出保存成功的提示
	        ToastUtils.showShortToast(getContext(), R.string.save_successfully);
	
	        // 完成了持久化之后的其他操作
	        afterSaveOrUpdate();
	
	        return true;
	    }
	}

在这份代码中，我们将顶层的类设计成一个抽象类。这样的好处在于，如果我们希望子类必须实现一个方法，那么我们可以将该方法定义成抽象的；如果我们要求子类不必实现某个方法，但是可以选择性地实现某个方法，我们可以将该方法定义成空的方法。

以上是保存操作的基本流程，我们首先要求子类必须实现输入前校验的方法，只有当校验成功的时候才继续进行操作，我们将它交给子类实现是因为不同的实体可能对校验有不同的要求。然后我们定义一个空的方法`beforeSaveOrUpdate()`，给用户提供可选的操作。它可以用来在真正对数据进行持久化之前添加一些操作。然后，进入了持久化操作层，在这里我们先要判断当前的实体是否是新的实体。在判断是否是新的的实体的时候，我们需要子类实现`getStoreOfModel()`和`getModel()`两个抽象方法，用来获取数据库对象和实体对象，然后我们可以用它进行判断。这里我们还是用了一个单例的`isNewModel`字段，并且在保存完毕之后还要将其手工置为false. 接下来是实际的插入和更新数据库的操作，这些方法我们交给子类来实现。接下来的一些操作属于模板的逻辑。最后，我们又加了一个`afterSaveOrUpdate()`方法，用于提供给用户进行一些保存之后的其他的逻辑处理。
	
以上就是我们设计的一种模板，它将程序的逻辑框架放在了顶层的类中，避免了重复和冗余的代码。同时，将整个流程设计成一种模板，当需要修改的逻辑的时候，只要修改顶层中的逻辑就可以了，也大大降低了维护的难度。

### 总结

上面是模板模式的两种使用示例，从这里可以看出，模板模式充分利用了面向对象的好处。通过使用模板模式，我们将业务逻辑抽象成一个框架，避免了冗余的代码，同时也降低了维护的难度。

下面是一些总结：

1. 模板方法定义了算法的步骤，吧这些步骤的实现延迟到了子类；
2. 模板方法模式为我们提供了一种代码复用的技巧；
3. 为了防止子类改变模板方法中的算法，我们可以将模板方法声明为final；
4. 从上面看出模板方法和策略模式是完全不同的，模板方法定义了一套业务逻辑的步骤，而策略方式仅仅对实现某个业务逻辑的策略有修改。