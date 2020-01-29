---
layout: post
title: RecyclerView基本使用
excerpt: RecyclerView基本使用
categories: Android
keywords: Android, RecyclerView
---
## 控件初始化
```java

    protected BleListAdapter mAdapter;
    protected RecyclerView.LayoutManager mLayoutManager;

    @ViewById
    RecyclerView mRecyclerView;

    @AfterViews
    void initView() {
        L.d("[BleListFragment] initView");
        initRecyclerView();
        BleManagerService.setOnScanResultListener(new BleManagerService.OnScanResultListener() {
            @Override
            public void onScanResultListener(String name, String addr, int rssi) {
                L.d("onScanResultListener");
                addDeviceToList(name, addr, rssi);
            }
        });
    }

    @Override
    public void showSearch() {
        pro_search.setVisibility(View.VISIBLE);
        mAdapter.clear();
    }

 /**
     * 配置RecyclerView
     */
    private void initRecyclerView() {
        mLayoutManager = new LinearLayoutManager(getActivity());
        mRecyclerView.setLayoutManager(mLayoutManager);
        mRecyclerView.addItemDecoration(new DividerItemDecoration(getActivity(), DividerItemDecoration.VERTICAL_LIST));
        mAdapter = new BleListAdapter();
        mAdapter.setOnItemClickListener(new BleListAdapter.OnItemClickListener() {
            @Override
            public void onClick(View v, int position, BleItem bleItem) {
                String bleName = bleItem.getBleName();
                String bleAddr = bleItem.getBleAddr();
                L.d("bleName: " + bleName + ",bleAddr: " + bleAddr);
                mPresenter.connectBle(bleName, bleAddr);
            }
        });
        mRecyclerView.setAdapter(mAdapter);
    }

    /**
     * 将设备添加到显示列表
     *
     * @param name
     * @param addr
     */
    @UiThread
    void addDeviceToList(String name, String addr, int rssi) {
        BleItem bleItem = new BleItem(name, addr, rssi);
        mAdapter.add(bleItem);
    }

```
此例中，列表的数据是通过扫描周围的设备动态添加进来的，RecyclerView的初始化在initRecyclerView中。

## Adapter
```java
public class BleListAdapter extends RecyclerView.Adapter<BleListAdapter.ViewHolder> {

    private List<BleItem> mDatas;
    private LayoutInflater inflater;

    private OnItemClickListener listener;

    public interface OnItemClickListener {
        void onClick(View v, int position, BleItem bleItem);
    }

    public void setOnItemClickListener(OnItemClickListener listener) {
        this.listener = listener;
    }

    public BleListAdapter() {
        mDatas = new ArrayList<>();
    }

    @Override
    public int getItemCount() {
        return mDatas.size();
    }

    public void add(BleItem bleItem){
        mDatas.add(bleItem);
        notifyDataSetChanged();
    }

    public void clear(){
        if (mDatas != null)
        {
            mDatas.clear();
            notifyDataSetChanged();
        }
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        inflater = LayoutInflater.from(parent.getContext());
        View view = inflater.inflate(R.layout.blelist_list, parent, false);
        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(final ViewHolder holder, final int position) {
        holder.tv_name.setText(mDatas.get(position).getBleName());
        holder.tv_addr.setText(mDatas.get(position).getBleAddr());
        holder.tv_rssi.setText(mDatas.get(position).getRssi() + " dBm");
    }

    class ViewHolder extends RecyclerView.ViewHolder {
        private TextView tv_name;
        private TextView tv_addr;
        private TextView tv_rssi;

        public ViewHolder(View itemView) {
            super(itemView);

            tv_name = (TextView) itemView.findViewById(R.id.tv_name);
            tv_addr = (TextView) itemView.findViewById(R.id.tv_addr);
            tv_rssi = (TextView) itemView.findViewById(R.id.tv_rssi);

            itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    int position = getPosition();
                    if (listener != null) {
                        listener.onClick(v, position, mDatas.get(position));
                    }
                }
            });
        }
    }
}
```
adapter的数据主要是通过add方法添加进来，简单修改下可直接通过构造方法将数据传递进来。
