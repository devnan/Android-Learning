----------
##Fragment生命周期
![](https://leanote.com/api/file/getImage?fileId=56bc1358ab64416aa0003d5c)

Fragment除了Activity有的方法外还提供了附加的方法
1.onAttach():当碎片和活动建立关联时调用
2.onCreateView():为碎片创建视图（加载布局）时调用
3.onActivityCreated():确保与碎片相关联的活动一定已经创建完毕的时候调用
4.onDestroyView():当与碎片关联的视图被移除的时候调用
5.onDetach():当碎片和活动解除关联的时候调用 fragment的生命周期

**Activity与Fragment的比较**
![](https://leanote.com/api/file/getImage?fileId=56bc1358ab64416aa0003d5b)

----------

##android.support.v4.app.Fragment和android.app.Fragment的区别
1.最低支持版本不同
android.app.Fragment 兼容的最低版本是android:minSdkVersion="11" 即3.0版
android.support.v4.app.Fragment 兼容的最低版本是android:minSdkVersion="4" 即1.6版

2.需要导jar包
fragment android.support.v4.app.Fragment 需要引入包android-support-v4.jar

3.在Activity中取的方法不同
android.app.Fragment使用getFragmentManager().findFragmentById()获得，继承Activity;
android.support.v4.app.Fragment使用getSupportFragmentManager().findFragmentById()获得，需要继承**FragmentActivity**

----------
##FragmentManager
为了管理Activity中的fragments，需要使用FragmentManager。
为了得到它，需要调用Activity中的getFragmentManager()方法。
因为FragmentManager的API是在Android 3.0，也即API level 11开始引入的，所以对于之前的版本，需要使用support library中的FragmentActivity，并且使用getSupportFragmentManager()方法。

----------


##Fragment Transactions
使用Fragment时，可以通过用户交互来执行一些动作，比如增加、移除、替换等。
所有这些改变构成一个集合，这个集合被叫做一个transaction。
可以调用FragmentTransaction中的方法来处理这个transaction，并且可以将transaction存进由activity管理的back stack中，这样就可以进行fragment变化的回退操作。
可以这样得到FragmentTransaction类的实例：　

    FragmentManager fragmentManager = getFragmentManager();
    FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

每个transaction是一组同时执行的变化的集合。
用add(), remove(), replace()方法，把所有需要的变化加进去，然后调用commit()方法，将这些变化应用。
在commit()方法之前，你可以调用addToBackStack()，把这个transaction加入back stack中去，这个back stack是由activity管理的，当用户按返回键时，就会回到上一个fragment的状态。
比如下面就是用一个新的fragment取代之前的fragment，并且将前次的状态存储在back stack中。

    // Create new fragment and transaction
    Fragment newFragment = new ExampleFragment();
    FragmentTransaction transaction = getFragmentManager().beginTransaction();
    // Replace whatever is in the fragment_container view with this fragment,
    // and add the transaction to the back stack
    transaction.replace(R.id.fragment_container, newFragment);
    transaction.addToBackStack(null);
    // Commit the transaction
    transaction.commit();
即动态添加碎片的步骤：
1.创建待添加的碎片实例。
2.获取到FragmentManager，在活动中可以直接调用getFragmentMananager()方法得到。
3.通过调用beginTransaction()开启事务
4.向容器内加入碎片，一般使用replace()方法实现，需要传入容器的id和待添加的碎片实例
5.提交事务，调用commit()方法来完成

----------


##**Activity与Fragment之间的通信**
**(1)Fragment调用Activity的数据**
通过在Activity中setArguments()以及在Fragment中getArguments()实现数据从Activity向Fragment传递

     public static ExampleFragment newInstance(String param) {
            ExampleFragment fragment = new ExampleFragment();
            //在Activity中创建Bundle数据包，并调用Fragment的setArguments(Bundle bundle)方法
            Bundle args = new Bundle();
            args.putString(ARG_PARAM, param);
            fragment.setArguments(args);
            return fragment;
    }
     
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getArguments() != null) {
            //在Fragment中通过getArguments().getString()获取Activity传来的值
            mParam1 = getArguments().getString(ARG_PARAM1);
        }
    }
**(2)Activity调用Fragment的数据**        
在Fragment中定义一个内部回调接口，让包含该Fragment的Activity实现该回调接口，这样Fragment可调用该回调方法将数据传递给Activity。下面主要实现在Activity调用Fragment的数据。
创建一个TestFragment类继承自Fragment，先在Fragment中定义一个内部回调接口：

     public interface OnFragmentInteractionListener {
        public void onFragmentInteraction(String uri);
    }
在Fragment的onAttach()方法中将Activity转换为OnFragmentInteractionListener接口：

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        try {
            mListener = (OnFragmentInteractionListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString()
                    + " must implement OnFragmentInteractionListener");
        }
    }
    
    private OnFragmentInteractionListener mListener;
    if (mListener != null) {
        mListener.onFragmentInteraction("需要传递给Activity的值");
    }

宿主Activity则需要实现此接口，并重写onFragmentInteraction(String uri)方法，这样就实现了数据从Fragment到Activity的传递：

    public class MainActivity extends ActionBarActivity implements TestFragment.OnFragmentInteractionListener
    @Override
    public void onFragmentInteraction(String uri) {
        TextView tv=(TextView)findViewById(R.id.tv);
        tv.setText(uri);
    }

----------


