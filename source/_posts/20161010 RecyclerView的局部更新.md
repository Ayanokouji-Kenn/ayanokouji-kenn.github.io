---
title: RecyclerView的局部更新
date: 2016-10-10 10:33:27
tags: Utils
---
我们知道recyclerView有好几个notify

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2524531-412ab4e561ac1f34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
除了最常用的`notifyDataSetChanged()`以外，还有下面的那些，知道position就可以进行更新了
但是！我得知道位置才能做到定向更新，那么我不想让所有都更新呢？
实际开发中其实经常有这种情况，可能只是调整一小部分，根本不需要整体全部刷一遍，下面进入正题
***
首先，项目里用到了
~~~java
    compile 'com.android.support:appcompat-v7:24.2.1'
    compile 'io.reactivex.rxjava2:rxjava:2.0.0-RC4'
    compile 'io.reactivex.rxjava2:rxandroid:2.0.0-RC1'
    compile 'com.android.support:recyclerview-v7:24.2.1'
~~~
当然rxjava不是必须的。。。不懂也没关系，这不是重点
上代码，其实就2个类，一个MainActivity，一个是Adapter
~~~java
public class MainActivity extends AppCompatActivity {
    private RecyclerView mRv;
    private Button mBtn;
    private List<Bean> mDatas;
    private DiffAdapter diffAdapter;
    private List<Bean> mNewDatas;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initView();
        initData();
        initEvent();
    }

    private void initView() {
        mRv = getView(R.id.rv);
        mBtn = getView(R.id.btn);
    }

    private void initData() {
        mDatas = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            //Bean就是个Bean类，里面就2个参数，一个id，一个name
            mDatas.add(new Bean(i, "name" + i));   
        }
    }

    private void initEvent() {
        mRv.setLayoutManager(new LinearLayoutManager(this));
        diffAdapter = new DiffAdapter(this, mDatas);
        mRv.setAdapter(diffAdapter);
        
        mBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                refresh();
            }
        });
    }

    private void refresh() {
        mNewDatas = new ArrayList<>();
        //模拟获取新数据，将原集合添加进来，删掉一个，添加一个
        mNewDatas.addAll(mDatas);                        
        mNewDatas.remove(0);
        Random random = new Random();
        int i = random.nextInt(10);
        mNewDatas.add(new Bean(i, "add" + i));
        //不懂rxjava的同学从这里
        Observable.fromArray(mDatas)                                       
               .map(new Function<List<Bean>, DiffUtil.DiffResult>() {
                    @Override
                    public DiffUtil.DiffResult apply(List<Bean> been) throws Exception { 
                        //到这里可以不用看，下面的才是关键 
                        //DiffUtil.calculateDiff（）是用来计算2个集合差异性的，返回了一个diffResult，下面会用到
                        return DiffUtil.calculateDiff(new DiffUtil.Callback() {                        
                            @Override
                            public int getOldListSize() {        //获取原集合大小
                                return mDatas == null ? 0 : mDatas.size();
                            }

                            @Override
                            public int getNewListSize() {    //获取新集合大小
                                return mNewDatas == null ? 0 : mNewDatas.size();
                            }

                            //判断2个item是否是同一个，逻辑根据实际开发需求来定，这里就用Bean.id来判断了
                            @Override
                            public boolean areItemsTheSame(int oldItemPosition, int newItemPosition) {
                                return mDatas.get(oldItemPosition).getId() == mNewDatas.get(newItemPosition).getId();
                            }
                            
                            //判断2个item的内容是否改变了，同上，这里用Bean.name来判断
                            @Override
                            public boolean areContentsTheSame(int oldItemPosition, int newItemPosition) {
                                return TextUtils.equals(mDatas.get(oldItemPosition).getName(), mNewDatas.get(newItemPosition).getName());
                            }

                            /*如果2个item是同一个，但是内容不同，即
                              areItemsTheSame 返回true
                              areContentsTheSame 返回了false*/
                            //那么会调用这个方法
                            @Nullable
                            @Override
                            public Object getChangePayload(int oldItemPosition, int newItemPosition) {
                                Bean oldBean = mDatas.get(oldItemPosition);
                                Bean newBean = mNewDatas.get(newItemPosition);
                                //将新旧2个item对象的不同之处返回，如果没有就返回null
                               /*这个例子里面反正就一个name不同，就直接返回String了，
                                如果变化的东西多，可以返回List,Map,SparseArray,bundle等等等等*/
                                if (!TextUtils.equals(oldBean.getName(), newBean.getName())) {        
                                    return newBean.getName();
                                } else {
                                    return null;
                                }
                            }
                        });
                    }
                })
                .subscribeOn(Schedulers.computation())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<DiffUtil.DiffResult>() {
                    @Override
                    public void accept(DiffUtil.DiffResult diffResult) throws Exception {
                    //上面的DiffUtil.calculateDiff（）返回的diffResult这里用到了
                        diffResult.dispatchUpdatesTo(diffAdapter);
                        mDatas = mNewDatas;
                        diffAdapter.setData(mDatas);
                    }
                });
        
    }
    
    private <T extends View> T getView(int id) {
        return (T)findViewById(id);
    }
}
~~~
~~~java
public class DiffAdapter extends RecyclerView.Adapter<DiffAdapter.DiffVh> {
    private List<Bean> mDatas;
    private Context mContext;
    public DiffAdapter(Context context, List<Bean> datas) {
        mContext = context;
        mDatas = datas;
    }

    @Override
    public DiffAdapter.DiffVh onCreateViewHolder(ViewGroup parent, int viewType) {
        
        return new DiffVh(LayoutInflater.from(mContext).inflate(R.layout.item_rv,parent,false));
    }

    @Override
    public void onBindViewHolder(DiffAdapter.DiffVh holder, int position) {
        holder.tvTitle.setText(mDatas.get(position).getId()+"");
        holder.tvDesc.setText(mDatas.get(position).getName());
    }

    @Override
    public void onBindViewHolder(DiffAdapter.DiffVh holder, int position, List<Object> payloads) {
        if (payloads.isEmpty()) {
            onBindViewHolder(holder,position);
        }else {
            holder.tvDesc.setText((CharSequence) payloads.get(0));
        }
    }

    @Override
    public int getItemCount() {
        return mDatas==null?0:mDatas.size();
    }

    public void setData(List<Bean> mDatas) {
        this.mDatas = mDatas;
    }

    class DiffVh extends RecyclerView.ViewHolder {

        TextView tvTitle;
        TextView tvDesc;

        DiffVh(View itemView) {
            super(itemView);
            tvTitle = (TextView) itemView.findViewById(R.id.tv_title);
            tvDesc = (TextView) itemView.findViewById(R.id.tv_desc);
        }
    }
}
~~~
布局文件不贴了吧   
`R.layout.item_rv`就是2个textview用来显示Bean的id和name
`R.layout.activity_main`就是一个recyclerview和一个button用来刷新而已
OK，到此为止，怎么用都在注释里