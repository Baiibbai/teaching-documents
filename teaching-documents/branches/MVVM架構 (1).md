
# MVVM架構

## 介紹

- 分成Model,ViewModel,View

    >Model : 邏輯
    >ViewModel : 控制View連結Activity/Fragment
    >View : 在Activity/Fragment裡，只保留單純的View

## 範例

- EventsFragment發Request將資料更新到RecyclerView
- 向下拉更新資料
- 點擊Item開啟新頁面

## 範例原始檔案



### MainActivity.java

```java=
public class MainActivity extends AppCompatActivity
{
	//FragmentTabHost
	private FragmentTabHost mTabHost;
	
	//textView
	private TextView mTitle, mContext;
	
	//佈局填充器
	private LayoutInflater mLayoutInflater;
	
	//tab選項文字
	private String[] mTextArray = { "Events","Coupon","Voucher","Discover","Setting" };
	
	//儲存tabhost的view
	private View[] mViewArray = new View[5];
	
	//每個tab對應的fragment
	private Class[] mFragmentArray = {EventsFragment.class,CouponFragment.class,VoucherFragment.class,DiscoverFragment.class,SettingFragment.class};

	@Override
	protected void onCreate(Bundle savedInstanceState)
	{
        super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	
		VolleyHelper.createInstance(getApplicationContext());
		ImageHelper.createInstance(getApplicationContext());
	
		//初始化
		initView();
	}
	
    //初始化元件
	public void initView()
    {
		mLayoutInflater = LayoutInflater.from(this);
		//找到TabHost
		mTabHost = findViewById(android.R.id.tabhost);
		mTabHost.setup(this, getSupportFragmentManager(), R.id.main_tabcontent);
		mTabHost.clearAllTabs();
		//去除分割線
		mTabHost.getTabWidget().setDividerDrawable(null);
		//得到fragment個數
		int count = mFragmentArray.length;
		//依照fragment的數量 設定切換按鈕
		for( int i = 0; i < count; i++ )
		{
			//儲存View的陣列
			mViewArray[i] = getTabItemView(i);
			//設置各Tab按鈕圖片文字
			TabHost.TabSpec tabSpec = mTabHost.newTabSpec(mTextArray[i]).setIndicator(mViewArray[i]);
			//添加對應的fragment
			mTabHost.addTab(tabSpec, mFragmentArray[i], null);
			//自定義click事件
			mTabHost.getTabWidget().getChildTabViewAt(i).setOnClickListener(new IndexClickListener(i));
		}
		//預選第一個按鈕
		mTabHost.setCurrentTab(0);
	}
	
	//設定tab按鈕文字圖標
	private View getTabItemView(int index)
	{
		View view = mLayoutInflater.inflate(R.layout.layout_tab_item_view, null);
		ImageView iv = view.findViewById(R.id.tab_item_iv_icon);
		switch( index )
		{
			case ApiDetail.ItemFragment.EVENTS:
				iv.setImageResource(R.drawable.main_bottom_iamge_selector);
				break;
			case ApiDetail.ItemFragment.COUPON:
				iv.setImageResource(R.drawable.main_bottom_image_selector1);
				break;
			case ApiDetail.ItemFragment.VOUCHER:
				iv.setImageResource(R.drawable.main_bottom_image_selector2);
				break;
			case ApiDetail.ItemFragment.DISCOVER:
				iv.setImageResource(R.drawable.main_bottom_image_selector3);
				break;
			case ApiDetail.ItemFragment.SETTING:
				iv.setImageResource(R.drawable.main_bottom_image_selector4);
				break;
		}
		return view;
	}
	
	//tabhost按鈕事件
	private class IndexClickListener implements View.OnClickListener
	{
		private int index;
		
		private IndexClickListener(int i)
		{
			index = i;
		}
		
		@Override
		public void onClick(View view)
		{
				//切換按扭動作
				mTabHost.setCurrentTab(index);
		}
	}
}
```

## activity_main.xml

```java=
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	xmlns:app="http://schemas.android.com/apk/res-auto"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical"
	tools:context=".MainActivity">
	
	<!--頁面內容-->
	<FrameLayout
	    android:id="@+id/main_tabcontent"
	    android:layout_width="fill_parent"
	    android:layout_height="0dp"
	    android:layout_weight="1"
	    android:background="@drawable/system_background_blur"
	    tools:visibility="gone"
	/>
	
	<!--菜單-->
	<androidx.fragment.app.FragmentTabHost
		android:id="@android:id/tabhost"
		android:layout_width="fill_parent"
		android:layout_height="70dp">
	</androidx.fragment.app.FragmentTabHost>
	
	<FrameLayout
	    android:id="@android:id/tabcontent"
	    android:layout_width="0dp"
	    android:layout_height="0dp"
	    android:layout_weight="0"
	/>
</LinearLayout>
```

## EventsFragment.java

```java=
public class EventsFragment extends Fragment
{
	//設定進度條
	private ProgressBar mProgressBar;
	//設定recyclerview
	private RecyclerView mRecyclerView;
	//設定RecyclerView.Adapter
	private RecyclerView.Adapter mRecyclerAdapter;
	//設定儲存資料陣列
	private List<EventsApi.ListEvents> mData = new ArrayList<>();
	//設定向下滑動更新
	private SwipeRefreshLayout mLaySwipe;
	
	@Nullable
	@Override
	public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState)
	{
		View v = inflater.inflate(R.layout.fragment_events, container, false);
		//綁定Recyclerview
		mRecyclerView = v.findViewById(R.id.events_recyclerview_events_list);
		//綁定進度條
		mProgressBar = v.findViewById(R.id.events_pgb);
		//向下拉更新
		mLaySwipe = v.findViewById(R.id.events_layswipe_update_update);
		//設定顏色
		mLaySwipe.setColorSchemeResources(android.R.color.holo_red_light, android.R.color.holo_blue_light,android.R.color.holo_green_light, android.R.color.holo_orange_light);
		//當動作被執行
		mLaySwipe.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener()
		{
			@Override
			public void onRefresh()
			{
				//取得店家資訊
				getStoreMessage();
				mRecyclerAdapter.notifyDataSetChanged();
				mLaySwipe.setRefreshing(false);
			}
		});
		getStoreMessage();
		return v;
	}
	
	//設定RecyclerView的Adapter
	@SuppressLint("WrongConstant")
	private void setRecyclerView()
	{
		Log.i("EventsFragment", "setRecyclerView");
		mRecyclerView.setLayoutManager(new LinearLayoutManager(getActivity()));
		mRecyclerAdapter = new EventsAdapter(mData, mItemClickListener);
		mRecyclerView.setAdapter(mRecyclerAdapter);
		mProgressBar.setVisibility(View.GONE);
		mLaySwipe.setVisibility(View.VISIBLE);
		mRecyclerView.setVisibility(View.VISIBLE);
	}
	
	//抓取資料
	private void getStoreMessage()
	{
		EventsRequest request = new EventsRequest(mEventsListenr);
		request.send();
	}
	
	//使用介面抓取資料
	private EventsRequest.EventsListener mEventsListenr = new EventsRequest.EventsListener()
	{
		@Override
		public void onResponse(List<EventsApi.ListEvents> result)
		{
			Log.i("EventsFragment", "eventsListener onResponse");
			mData = result;
			if( mData.size() != 0 )
			{
				setRecyclerView();
			}
			else
			{
				Log.i("EventsFragment", "eventsListener onResponse data is zero");
				Toast.makeText(getActivity(),"getData : zero",Toast.LENGTH_LONG).show();
			}
		}
		@Override
		public void onError(String errorResulte)
		{
			Log.i("EventsFragment", "eventsListener onError");
			Log.i("events", errorResulte);
			Toast.makeText(getActivity(), getString(R.string.getdata_error), Toast.LENGTH_LONG).show();
			mProgressBar.setVisibility(View.GONE);
		}
	};

	//按鈕事件
	private EventsAdapter.OnClickListener mItemClickListener = new EventsAdapter.OnClickListener()
	{
		@Override
		public void onClick(String url)
		{
			//設定bundle
			Bundle args = new Bundle();
			args.putString(ApiDetail.ItemName.ITEM_URL, url);
			FragmentTransaction transaction = (getActivity()).getSupportFragmentManager().beginTransaction();
			String fragmentTag = EventsDetailFragment.class.getSimpleName();
			Fragment newFragment = new EventsDetailFragment();
			
			//開啟內容
			if( newFragment != null )
			{
				Log.i("oldFragment", "popBack");
				FragmentManager fm = getActivity().getSupportFragmentManager();
				fm.popBackStack(fragmentTag, FragmentManager.POP_BACK_STACK_INCLUSIVE);
			}
			Log.i("newFragment", "addFragment");
			if( args != null )
			{
				newFragment.setArguments(args);
			}
			transaction.add(R.id.main_tabcontent, newFragment, fragmentTag);
			transaction.addToBackStack(fragmentTag);
			transaction.commit();
		}
	};
}
```

## fragment_events.xml

```xml=
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical"
	android:background="@drawable/system_background_blur"
>
	<ProgressBar
		android:id="@+id/events_pgb"
		style="?android:attr/progressBarStyle"
		android:layout_width="50dp"
		android:layout_height="50dp"
		android:layout_gravity="center"
		android:paddingTop="10dp"
	/>
	<androidx.swiperefreshlayout.widget.SwipeRefreshLayout
		android:id="@+id/events_layswipe_update_update"
		android:layout_width="match_parent"
		android:layout_height="0dp"
		android:layout_weight="1"
		android:visibility="gone">
		
		<androidx.recyclerview.widget.RecyclerView
			android:id="@+id/events_recyclerview_events_list"
			android:layout_width="match_parent"
			android:layout_height="wrap_content"
			android:clipToPadding="false"
			android:visibility="gone"
			android:scrollbars="none"
			tools:visibility="visible"
		/>
	</androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
</LinearLayout>
```

## EventsAdapter.java

```java=
public class EventsAdapter extends RecyclerView.Adapter<EventsAdapter.ViewHolder>
{
	//儲存資料
	public List<EventsApi.ListEvents> mData;
	//清單按鈕
	private OnClickListener mListener;
	//按鈕事件
	//改為傳送點擊位置 將邏輯拋回fragment
	public interface OnClickListener
	{
		void onClick(String url);
	}
	class ViewHolder extends RecyclerView.ViewHolder
	{
		//商店名稱
		TextView mTvStoreName;
		//開始時間
		TextView mTvBeginTime;
		//結束時間
		TextView mTvEndTime;
		//商店圖片
		ImageView mIvStorePic;
		
		ViewHolder(@NonNull View v)
		{
			super(v);
			//商店名稱
			mTvStoreName = v.findViewById(R.id.events_rcv_tv_title);
			//開始時間
			mTvBeginTime = v.findViewById(R.id.events_rcv_tv_begin_date);
			//結束時間
			mTvEndTime = v.findViewById(R.id.events_rcv_tv_end_date);
			//商店圖片
			mIvStorePic = v.findViewById(R.id.events_rcv_iv_icon_url);
		}
	}
	
	//資料連結
	public EventsAdapter(List<EventsApi.ListEvents> info, OnClickListener listener)
	{
		mData = info;
		mListener = listener;
	}
	
	@NonNull
	@Override
	public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType)
	{
		//設定 ViewHolder
		View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.layout_events_recyclerview, parent, false);
		return new ViewHolder(v);
	}

	@Override
	public void onBindViewHolder(@NonNull final ViewHolder holder, final int position)
	{
		//加載圖片
		final ImageLoader.ImageListener imageListener =ImageLoader.getImageListener(holder.mIvStorePic, R.drawable.event_list_0001, R.drawable.system_input_wrong);
		//從網路上加載圖片
		ImageHelper.getInstance().loadingImage(mData.get(position).iconUrl, imageListener);
		//將資料填入recyclerview
		holder.mTvStoreName.setText(mData.get(position).title);
		holder.mTvBeginTime.setText(mData.get(position).beginData);
		holder.mTvEndTime.setText(mData.get(position).endData);
		//點擊項目觸發按鈕
		holder.itemView.setOnClickListener(new View.OnClickListener()
		{
		@Override
		public void onClick(View view)
		{
			mListener.onClick(mData.get(position).url);
		}
	    });
	
	    //點擊項目觸發按鈕
	    holder.itemView.setOnClickListener(new View.OnClickListener()
	    {
	    	@Override
		    public void onClick(View view)
		    {
			    mListener.onClick(mData.get(position).url);
		    }
	    });
    }
    
	//清單長度
	@Override
	public int getItemCount()
	{
		return mData.size();
	}
}
```

## layout_events_recyclerview.xml

```html=
<LinearLayout
		xmlns:android="http://schemas.android.com/apk/res/android"
		xmlns:tools="http://schemas.android.com/tools"
		xmlns:app="http://schemas.android.com/apk/res-auto"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:orientation="vertical"
		android:gravity="center"
		android:background="@color/translucent"
		tools:background="@drawable/system_background_blur"
		tools:layout_height="match_parent"
	>
	
		<LinearLayout
			android:layout_width="match_parent"
			android:layout_height="wrap_content"
			android:orientation="horizontal"
			android:gravity="center"
			android:background="@drawable/system_list_btn"
			android:paddingBottom="20dp"
			android:paddingTop="20dp"
			android:baselineAligned="false">
			
			<LinearLayout
				android:layout_width="0dp"
				android:layout_height="match_parent"
				android:layout_weight="2"
				android:gravity="center"
			>
				<ImageView
					android:id="@+id/events_rcv_iv_icon_url"
					android:layout_width="match_parent"
					android:layout_height="match_parent"
					android:gravity="center"
				/>
			</LinearLayout>
			
			<LinearLayout
				android:layout_width="0dp"
				android:layout_height="wrap_content"
				android:layout_weight="5"
				android:orientation="vertical"
				android:gravity="center"
			>
				<TextView
					android:id="@+id/events_rcv_tv_title"
					android:layout_width="wrap_content"
					android:layout_height="wrap_content"
					android:textSize="@dimen/special_price"
					android:textColor="@color/events_title"
					android:textStyle="bold"
					android:paddingBottom="10dp"
					tools:text="EXTRA BOUNS !"
				/>
				<LinearLayout
					android:layout_width="wrap_content"
					android:layout_height="wrap_content"
					android:orientation="horizontal"
					android:gravity="center"
				>
					<TextView
						android:id="@+id/events_rcv_tv_begin_date"
						android:layout_width="wrap_content"
						android:layout_height="wrap_content"
						android:textSize="@dimen/textsize"
						android:textColor="@color/events_date"
						tools:text="04/10/2015"
					/>
					<TextView
						android:layout_width="wrap_content"
						android:layout_height="wrap_content"
						android:textSize="20sp"
						android:textColor="@color/events_date"
						android:text="@string/date"
					/>
					<TextView
						android:id="@+id/events_rcv_tv_end_date"
						android:layout_width="wrap_content"
						android:layout_height="wrap_content"
						android:textSize="@dimen/textsize"
						android:textColor="@color/events_date"
						tools:text="04/12/2015"
					/>
				</LinearLayout>
			</LinearLayout>
			
			<LinearLayout
				android:layout_width="30dp"
				android:layout_height="match_parent"
				android:gravity="center"
			>
				<ImageView
					android:layout_width="wrap_content"
					android:layout_height="wrap_content"
					android:layout_gravity="center"
					android:src="@drawable/system_icon_nextpage"
				/>
			</LinearLayout>
		</LinearLayout>
		
		<LinearLayout
			android:layout_width="match_parent"
			android:layout_height="wrap_content">
			<ImageView
				android:layout_width="match_parent"
				android:layout_height="wrap_content"
				android:background="@drawable/system_list_divider" />
		</LinearLayout>
	</LinearLayout>
```

## EventsDetailFragment.java

```html=
public class EventsDetailFragment extends Fragment
{
	@Nullable
	@Override
	public View onCreateView(
			@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState)
	{
		View v = inflater.inflate(R.layout.fragment_events_detail, container, false);
		ImageView imageView;
		imageView = v.findViewById(R.id.events_detail_iv_pic);
		//照片載入
		final ImageLoader.ImageListener imageListener =ImageLoader.getImageListener(imageView, R.drawable.event_list_0001, R.drawable.system_input_wrong);
		//抓取bundle資料
		Bundle bundle = getArguments();
		ImageHelper.getInstance().loadingImage(bundle.getString(ApiDetail.ItemName.ITEM_URL), imageListener);
		imageView.setClickable(true);
		return v;
	}
}
```

## fragment_events_detail.xml

```html=
<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout
		xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto"
		android:layout_width="fill_parent"
		android:layout_height="fill_parent"
		android:orientation="vertical"
		android:background="@drawable/system_background_blur"
	>
		<ImageView
			android:id="@+id/events_detail_iv_pic"
			android:layout_width="fill_parent"
			android:layout_height="match_parent"
			android:scaleType="fitXY"
		/>
	</LinearLayout>
```

# DataBinding

- build.gradle(app)
    - Data Binding 將View跟ViewModel綁定
    - 可在ViewModel做資料更改 直接設定給View
    - 在build.gradle(app)啟用dataBinding
 
```java=
android{
    comileSdkVersion 29
    buildToolsVersion "29.0.0"
    //啟用 Data Binding
    dataBinding{
        enabled = true;
    }
    ...
}
```
    
在原本的 fragment_events.xml 的最外層加上 (在 Root View 按 Alt + Enter -> Convert to data binding layout)，並在 裡面宣告要跟 View 綁定的 ViewModel

## fragment_events.xml

```html=
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:tools="http://schemas.android.com/tools">
	<data>
		<import type="android.view.View"/>
		<variable name="viewModel" type="com.example.mvvmtest2.EventsViewModel" />
	</data>
	...
</layout>	
```

## EventsFragment.java

- 宣告Binding與EventsViewModel.java
- 對View進行Binding 即會自動生成 FragmentEventsBinding此名稱

```java=
//Events DataBinding
private FragmentEventsBinding mBinding;
private EventsViewModel mViewModel;
```

- 設定EventsFragment的View
- 綁定ViewModel
- 使用setLifecycleOwner需要在 gradle.app引入
    - implementation 'androidx.lifecycle:lifecycle-extensions:2.1.0'

```java=
@Nullable
@Override
public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState)
{
        //綁定Layout
		mBinding = DataBindingUtil.inflate(inflater, R.layout.fragment_events, container, false);
		View v = mBinding.getRoot();

        //設定ViewModel與Layout的綁定
		mViewModel = new EventsViewModel();
		mBinding.setViewModel(mViewModel);
		//解決手機旋轉問題
		mBinding.setLifecycleOwner(this);
```

- 不需再宣告元件做綁定，直接使用mBinding找到元件
- 以下是綁定更改過後EventsFragment.java 的 OnCreate()

```java=
public class EventsFragment extends Fragment
{
	private FragmentEventsBinding mBinding;
	private EventsViewModel mViewModel;

	//設定RecyclerView.Adapter
	//private RecyclerView.Adapter mRecyclerAdapter;
	private EventsAdapter mRecyclerAdapter;

	//此頁面
	private Fragment mEventsFragment = this;

	@Nullable
	@Override
	public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState)
	{
		mBinding = DataBindingUtil.inflate(inflater, R.layout.fragment_events, container, false);
		View v = mBinding.getRoot();

		mViewModel = new EventsViewModel();
		mBinding.setViewModel(mViewModel);
		mBinding.setLifecycleOwner(this);

		//設定顏色
		mBinding.eventsLayswipeUpdateUpdate
				.setColorSchemeResources(android.R.color.holo_red_light, android.R.color.holo_blue_light,
				                         android.R.color.holo_green_light, android.R.color.holo_orange_light);
		//其餘設定...
		
		return view;
	}
```

# 重構EventsFragment.java

## EventsFragment.java

- 將EventsFragment.java 分割出 ViewModel(View資料直接設定)Model(邏輯)
- 原先寫在EventsFragemt.java的 request請求邏輯放到Model
- 將與此方法有關的邏輯都分割到Model

```java=
//請求request
private void getStoreMessage()
{
    EventsRequest request = new EventsRequest(mEventsListener);
    request.send();
}
```

## EventsModel.java

- 新增EventsModel.java
- 將發出request的邏輯搬到這裡
- 並設定接口，可將資料傳出

```java=
class EventsModel
{
	//Events資料抓取指向
	private itemEvents mResponse;
	
	//設定傳送接口
	public interface itemEvents
	{
		void onSuccess(List<EventsApi.ListApi> data);
		void onError(String errorResulte);
	}
	
	//抓取EventsList接口
	void sendMsgResponse(final itemEvents response){
		mResponse = response;
		EventsRequest request = new EventsRequest(mEventsListener);
		request.send();
	}
	
	private EventsRequest.EventsListener mEventsListener = new EventsRequest.EventsListener()
	{
		@Override
		public void onResponse(List<EventsApi.ListEvents> result)
		{
			mResponse.onSuccess(result);
			Log.i("EventsModel", "success");
		}

		@Override
		public void onError(String errorResulte)
		{
			mResponse.onError(errorResulte);
			Log.i("EventsModel", "error");
		}
	};
}
```

## EventsViewModel.java

- 新增EventsViewModel.java
- 將model抓到的request結果，利用接口在EventsViewModel.java上使用，設定給List
- 新增MutableLiveData<Boolean>變數，如果資料請求成功，將Boolean設定成true
    - MutableLiveData<> 可設定Observe 如資料變動 即可呼叫監聽器執行動作
    - 使用方法:要將資料設定進去 使用postvalue()

- 在EventsFragment.java設定監聽此變數判斷是否抓取List成功

```java=
public class EventsViewModel extends ViewModel
{
	//List
	List<EventsApi.ListEvents> mEventsList = new ArrayList();
	//model
	private EventsModel mModel = new EventsModel();
	//是否抓取到資料
	MutableLiveData<Boolean> mIsEventsDataGet = new MutableLiveData<>();
	
	//抓取EventsList Request
	void getEventsList()
	{
		mModel.sendMsgResponse(new EventsModel.itemEvents()
		{
			@Override
			public void onSuccess(List<EventsApi.ListEvents> data)
			{
				//資料抓取成功
				mIsEventsDataGet.postValue(true);
				//將資料放入List
				mEventsList = data;
				
				Log.i("EventsViewModel", "getEventsList:onSuccess");
			}

			@Override
			public void onError(String errorResulte)
			{
				//資料抓取失敗
				mIsEventsDataGet.postValue(false);
				Log.i("EventsViewModel", "getEventsList:onError");
			}
		});
	}
}
```

- 在onCreate()裡呼叫ViewModel的request請求(getEventsList())
- 並監聽MutableLiveData<Boolean> mIsEventsDataGet是否成功抓取List，並設定Adapter

```java=
        mViewModel.getEventsList();
		mViewModel.mIsEventsDataGet.observe(this, new Observer<Boolean>()
		{
			@Override
			public void onChanged(Boolean aBoolean)
			{
				if( aBoolean )
				{
					//mData = mViewModel.mEventsList;
					setRecyclerView();
				}
			}
		});

		mViewModel.mEventsList.observe(this, new Observer<List<EventsInfo>>()
		{
			@Override
			public void onChanged(List<EventsInfo> listEvents)
			{
				try
				{
					Log.i("EventsFragment", "eventsList:observe");
					mRecyclerAdapter.refresh(listEvents);
					mRecyclerAdapter.notifyDataSetChanged();
				}
				catch( Exception e )
				{
					e.printStackTrace();
				}
			}
		});

		//當動作被執行
		mBinding.eventsLayswipeUpdateUpdate.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener()
		{
			@Override
			public void onRefresh()
			{
				//取得店家資訊
				mViewModel.getEventsList();
				mRecyclerAdapter.notifyDataSetChanged();
				mBinding.eventsLayswipeUpdateUpdate.setRefreshing(false);
			}
		});
```

# 改寫Adapter

## 新增Comparable.java(interface)

- 可對資料做比對，用於更新機制

```java=
public interface Comparable<T extends Compable<T>>
{
    boolean areItemsTheSame(T t);
    boolean areContentsTheSame(T t);
}
```

## 新增資料EventsInfo.java

- 使資料可以做比對、更新
- 將原本的EventsApi.ListEvents裡儲存的資料型態都放進去

```java=
public class EventsInfo implements Comparable<EventsInfo>
{
	public String title, beginData, endData,iconUrl,url;

	@Override
	public boolean areItemsTheSame(EventsInfo eventsInfo)
	{
		return this.title.equals(eventsInfo.title);
	}

	@Override
	public boolean areContentsTheSame(EventsInfo eventsInfo)
	{
		if(this.iconUrl.equals(eventsInfo.iconUrl)){
			return false;
		}

		if(this.url.equals(eventsInfo.url)){
			return false;
		}

		if(!this.beginData.equals(eventsInfo.beginData)){
			return false;
		}

		return this.endData.equals(eventsInfo.endData);
	}
}
```

## layout_events_recyclerview.xml

- 對layout_events_recyclerview.xml做DataBinding

```html=
<layout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:tools="http://schemas.android.com/tools"
		xmlns:app="http://schemas.android.com/apk/res-auto">
	<data>
		<variable name="item" type="com.example.mvvmtest2.EventsInfo" />
	</data>
	...
</layout>
```

## EventsAdapter.java

- 使此Adapter可以共用
- 點擊傳出點擊位置
- 有更新機制
- 可以用DataBinding

```java=
public class EventsAdapter extends RecyclerView.Adapter<EventsAdapter.ViewHolder>
{
    //儲存資料
    //public List<EventsApi.ListEvents> mData;
	//宣告可比對資料的 List
	private List<? extends Comparable> mItemList;
	//RecyclerView的Layout
	private int mLayoutRes;

	//清單按鈕
	private OnClickListener mListener;

	//按鈕事件
	//改為傳送點擊位置 將邏輯拋回fragment
	public interface OnClickListener
	{
		void onClick(int position);
	}

	class ViewHolder extends RecyclerView.ViewHolder
	{
	    //因為使用DataBinding 所以不用額外宣告元件
	    //商店名稱
		//TextView mTvStoreName;
		//開始時間
		//TextView mTvBeginTime;
		//結束時間
		//TextView mTvEndTime;
		//商店圖片
		//ImageView mIvStorePic;
				
		private ViewDataBinding mItemBinding;

		ViewHolder(ViewDataBinding binding)
		{
			super(binding.getRoot());
			mItemBinding = binding;
		}

		void bind(Comparable item)
		{
			mItemBinding.setVariable(BR.item, item);
			mItemBinding.executePendingBindings();
		}

	}


	//資料連結
	public EventsAdapter(List<? extends Comparable> itemList, int layoutRes, OnClickListener listener)
	{
	    //Adapter輸入資料更改
		mItemList = itemList;
		mLayoutRes = layoutRes;
		mListener = listener;
	}

	@NonNull
	@Override
	public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType)
	{
	    //這邊更改為用Binding的方法 塞入Layout
		LayoutInflater layoutInflater = LayoutInflater.from(parent.getContext());
		ViewDataBinding itemBinding = DataBindingUtil.inflate(layoutInflater, mLayoutRes, parent, false);

		return new ViewHolder(itemBinding);
	}

    
	@Override
	public void onBindViewHolder(@NonNull final ViewHolder holder, final int position)
	{
	    //加載圖片在layout另外做設定 也不需要綁訂元件
	    
	    //將Adapter與List綁定
		Comparable item = mItemList.get(position);
		holder.bind(item);
        
        //點擊事件抓取position
		holder.itemView.setOnClickListener(new View.OnClickListener()
		{
			@Override
			public void onClick(View view)
			{
				mListener.onClick(position);
				Log.i("CommonAdapter", "position:" + position);
			}
		});

	}

	//清單長度
	@Override
	public int getItemCount()
	{
		return mItemList.size();
	}

    //在此新增更新功能
	public void refresh(List<? extends Comparable> itemList)
	{
		DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(new ItemDiffCallBack(mItemList, itemList), true);
		diffResult.dispatchUpdatesTo(this);
		mItemList = itemList;
	}

    //使新舊資料做比較 更新
	public class ItemDiffCallBack extends DiffUtil.Callback
	{
		private List<? extends Comparable> mOldItemList;
		private List<? extends Comparable> mNewItemList;

		ItemDiffCallBack(List<? extends Comparable> oldItemList, List<? extends Comparable> newItemList)
		{
			this.mOldItemList = oldItemList;
			this.mNewItemList = newItemList;
		}

		@Override
		public int getOldListSize()
		{
			return mOldItemList != null ? mOldItemList.size() : 0;
		}

		@Override
		public int getNewListSize()
		{
			return mNewItemList != null ? mNewItemList.size() : 0;
		}

		@SuppressWarnings("unchecked")
		@Override
		public boolean areItemsTheSame(int oldItemPosition, int newItemPosition)
		{
			return mOldItemList.get(oldItemPosition).areItemsTheSame(mNewItemList.get(newItemPosition));
		}

		@SuppressWarnings("unchecked")
		@Override
		public boolean areContentsTheSame(int oldItemPosition, int newItemPosition)
		{
			return mOldItemList.get(oldItemPosition).areContentsTheSame(mNewItemList.get(newItemPosition));
		}
	}
}
```

## EventsFragment.java

- 因為Adapter已經更改了，所以下一步改寫setRecyclerView

```java=
    //設定RecyclerView的Adapter
	@SuppressLint("WrongConstant")
	private void setRecyclerView()
	{
		Log.i("EventsFragment", "setRecyclerView");
		ArrayList<EventsInfo> infoList = new ArrayList<>();
		mBinding.eventsRecyclerviewEventsList.setLayoutManager(new LinearLayoutManager(getActivity()));
		mRecyclerAdapter = new EventsAdapter(infoList, R.layout.layout_events_recyclerview, mItemClickListener);
		mBinding.eventsRecyclerviewEventsList.setAdapter(mRecyclerAdapter);
		
        mBinding.eventsPgb.setVisibility(View.GONE);
        mBinding.eventsLayswipeUpdateUpdate.setVisibility(View.VISIBLE);
        mBinding.eventsRecyclerviewEventsList.setVisibility(View.VISIBLE);

		//分割線
		DividerItemDecoration mDivider = new DividerItemDecoration(getActivity(), DividerItemDecoration.VERTICAL);
		mDivider.setDrawable(getResources().getDrawable(R.drawable.system_list_divider));
		if( mBinding.eventsRecyclerviewEventsList.getItemDecorationCount() == 0 )
		{
			mBinding.eventsRecyclerviewEventsList.addItemDecoration(mDivider);
		}
	}
```

## RecyclerView分割線

- 因為每更新一次資料，RecyclreView的item之間間隔會越來越大，為了解決這個問題，所以要先判斷分割線是否已經被新增過(是否為0)，這樣才不會重複新增

```java=
	    //分割線
		DividerItemDecoration mDivider = new DividerItemDecoration(getActivity(), DividerItemDecoration.VERTICAL);
		mDivider.setDrawable(getResources().getDrawable(R.drawable.system_list_divider));
		if( mBinding.eventsRecyclerviewEventsList.getItemDecorationCount() == 0 )
		{
			mBinding.eventsRecyclerviewEventsList.addItemDecoration(mDivider);
		}
```

#改寫EventsFragment.java的按鈕邏輯

- 原先傳入值為url要改寫為position
- 將抓取圖片顯示事件寫到EventsViewModel.java

## EventsViewModel.java

- 宣告

```java=
    //點擊儲存字串
	private List<EventsInfo> mEventsClickList;

	//EventsDetail頁面圖片
	public MutableLiveData<String> mEventDetailImg = new MutableLiveData<>();

	//儲存點擊位置
	private int mPosition;
```

- 設定將點擊位置傳入的方法

```java=
    //設定click的位置
	void setClickPosition(int position)
	{
		this.mPosition = position;
		Log.i("EventsViewmodel", "position:" + position);
	}
```

- 利用傳入的位子，抓取position對應的url

```java=
    //利用Click位置抓取資料
	String getEventsClickImage()
	{
		Log.i("EventsViewModel", "getEventsClickImage :" + mEventsClickList.get(mPosition).url);
		return mEventsClickList.get(mPosition).url;
	}
```

## EventsFragment.java

- 改寫按鈕事件內容，使點擊設定Bundle為url

``` java=
    //按鈕事件
	private EventsAdapter.OnClickListener mItemClickListener = new EventsAdapter.OnClickListener()
	{
		@Override
		public void onClick(int position)
		{
			//將位置訊息傳入
			mViewModel.setClickPosition(position);
			//設定bundle
			Bundle args = new Bundle();
			args.putString(ApiDetail.ItemName.ITEM_URL, mViewModel.getEventsClickImage());

			FragmentTransaction transaction = (getActivity()).getSupportFragmentManager().beginTransaction();
			String fragmentTag = EventsDetailFragment.class.getSimpleName();
			Fragment newFragment = new EventsDetailFragment();

			newFragment.setArguments(args);
			//在此將原先的tabcontent改成新增的root_frame
			transaction.add(R.id.fragment_events_root_frame, newFragment, fragmentTag);
			//隱藏此頁面
			transaction.hide(mEventsFragment);
			//強迫新Fragment執行STARTED生命週期
			transaction.setMaxLifecycle(newFragment, Lifecycle.State.STARTED);

			transaction.addToBackStack(fragmentTag);
			transaction.commit();
		}
	};
```

# 使用Glis直接下載圖片

- 新增layout的Image設定方法
- 原先使用ImageHelper需要載入時，使用方法

```java=
public class ImageHelper
{
	private static ImageHelper mInstance;
	private ImageLoader mImageLoader;
	private ImageHelper()
	{
		//建立image的Request 建構式 只會建一次
		RequestQueue mRequestQueue;
		mRequestQueue = VolleyHelper.getInstance().getRequestQueue();
		mImageLoader = new ImageLoader(mRequestQueue, new BitmapCache());
	}
	public static void createInstance(Context context)
	{
		if(mInstance == null){
			mInstance = new ImageHelper();
		}
	}
	//呼叫即建立一個新的ImageHelper
	public static ImageHelper getInstance()
	{
		return mInstance;
	}
	//發生的事件
	public void loadingImage(String iconUrl, ImageLoader.ImageListener imageListener)
	{
		mImageLoader.get(iconUrl,imageListener);
	}
}
```

## gradle(app)

- 為了使用功能，使傳入url可以直接下載圖片
- 在gradle.app裡下載

```java=
implementation 'com.github.bumptech.glide:glide:4.8.0'
```

## ImageHelper.java

- 在ImageHelper.java裡新增方法
- 之後即可在layout裡用app:imageUrl設定

```java=
    @BindingAdapter({ "imageUrl"})
	public static void loadImage(ImageView iv, String url){
		Glide.with(iv.getContext()).load(url).into(iv);
	}
```

# RecyclerView的layout資料設定

## layout_events_recyclerview.xml

- 使用Binding剛加入的item資料
- 因為引入的是EventsInfo，所以可以直接做資料設定
- 變數名稱要打對，否則會Binding錯誤

```xml=
    <layout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:tools="http://schemas.android.com/tools"
		xmlns:app="http://schemas.android.com/apk/res-auto">
	<data>
		<variable name="item" type="com.example.mvvmtest2.EventsInfo" />

	</data>
	<LinearLayout
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:orientation="vertical"
		android:gravity="center"
		android:background="@color/translucent"
		tools:background="@drawable/system_background_blur"
		tools:layout_height="match_parent"
	>


		<LinearLayout
			android:layout_width="match_parent"
			android:layout_height="wrap_content"
			android:orientation="horizontal"
			android:gravity="center"
			android:background="@drawable/system_list_btn"
			android:paddingBottom="20dp"
			android:paddingTop="20dp"
			android:baselineAligned="false">

			<LinearLayout
				android:layout_width="0dp"
				android:layout_height="match_parent"
				android:layout_weight="2"
				android:gravity="center"
			>
                <!--在此有變更 直接傳入url請求網路圖片-->
				<ImageView
					android:id="@+id/events_rcv_iv_icon_url"
					android:layout_width="match_parent"
					android:layout_height="match_parent"
					android:gravity="center"
					app:imageUrl="@{item.iconUrl}"
				/>

			</LinearLayout>


			<LinearLayout
				android:layout_width="0dp"
				android:layout_height="wrap_content"
				android:layout_weight="5"
				android:orientation="vertical"
				android:gravity="center"
			>
			    <!--在此有變更 直接抓取item裡面資料-->
				<TextView
					android:id="@+id/events_rcv_tv_title"
					android:layout_width="wrap_content"
					android:layout_height="wrap_content"
					android:textSize="@dimen/special_price"
					android:textColor="@color/events_title"
					android:textStyle="bold"
					android:paddingBottom="10dp"
					tools:text="EXTRA BOUNS !"
					android:text="@{item.title}"
				/>

				<LinearLayout
					android:layout_width="wrap_content"
					android:layout_height="wrap_content"
					android:orientation="horizontal"
					android:gravity="center"
				>
				    <!--在此有變更 直接抓取item裡面資料-->
					<TextView
						android:id="@+id/events_rcv_tv_begin_date"
						android:layout_width="wrap_content"
						android:layout_height="wrap_content"
						android:textSize="@dimen/textsize"
						android:textColor="@color/events_date"
						tools:text="04/10/2015"
						android:text="@{item.beginData}"
					/>
					<TextView
						android:layout_width="wrap_content"
						android:layout_height="wrap_content"
						android:textSize="20sp"
						android:textColor="@color/events_date"
						android:text="@string/date"
					/>
					<TextView
						android:id="@+id/events_rcv_tv_end_date"
						android:layout_width="wrap_content"
						android:layout_height="wrap_content"
						android:textSize="@dimen/textsize"
						android:textColor="@color/events_date"
						tools:text="04/12/2015"
						android:text="@{item.endData}"
					/>

				</LinearLayout>


			</LinearLayout>

			<LinearLayout
				android:layout_width="30dp"
				android:layout_height="match_parent"
				android:gravity="center"
			>
				<ImageView
					android:layout_width="wrap_content"
					android:layout_height="wrap_content"
					android:layout_gravity="center"
					android:src="@drawable/system_icon_nextpage"
				/>
			</LinearLayout>
		</LinearLayout>
	</LinearLayout>
</layout>
```

# 啟動Adapter的更新機制

## EventsViewModel.java

- 將原先儲存的List改為MutableLiveData，使List內容變動時，可以被監聽

```java=
//List
MutableLiveData<List<EventsApi.ListEvents>> mEventsList = new MutableLiveData<>();
```

- 因為改為MutableLiveData 資料儲存要改為postValue

```java=
    //抓取EventsList Request
	void getEventsList()
	{
		mModel.sendMsgResponse(new EventsModel.itemEvents()
		{
			@Override
			public void onSuccess(List<EventsInfo> data)
			{
				//資料抓取成功
				mIsEventsDataGet.postValue(true);
				//將資料放入List
				mEventsList.postValue(data);

				mEventsClickList = data;
				Log.i("EventsViewModel", "getEventsList:onSuccess");
			}

			@Override
			public void onError(String errorResulte)
			{
				//資料抓取失敗
				mIsEventsDataGet.postValue(false);
				Log.i("EventsViewModel", "getEventsList:onError");
			}
		});
	}

```

## EventsFragment.java

- 因為將資料直接post到List上了，所以不需要額外儲存，更新時也會自動更新上去

```java=
        mViewModel.getEventsList();
		mViewModel.mIsEventsDataGet.observe(this, new Observer<Boolean>()
		{
			@Override
			public void onChanged(Boolean aBoolean)
			{
				if( aBoolean )
				{
					//mData = mViewModel.mEventsList;
					setRecyclerView();
				}
			}
		});
```

- 將Adapter宣告 改為剛改好的Adapter

```java=
//設定RecyclerView Adapter
//private RecyclerView.Adapter mRecyclerAdapter
private EventsAdapter mRecyclerAdapter;
```

- 將EventsApi.ListEvents都換成EventsInfo
- 當List資料變動會呼叫

```java=
mViewModel.mEventsList.observe(this, new Observer<List<EventsInfo>>()
		{
			@Override
			public void onChanged(List<EventsInfo> listEvents)
			{
				try
				{
					Log.i("EventsFragment", "eventsList:observe");
					mRecyclerAdapter.refresh(listEvents);
					mRecyclerAdapter.notifyDataSetChanged();
				}
				catch( Exception e )
				{
					e.printStackTrace();
				}
			}
		});
```

# View元件顯示

## EventsFragment.java

- 將Binding設定的顯示元件 改寫到ViewModel

```java=
//設定RecyclerView的Adapter
	@SuppressLint("WrongConstant")
	private void setRecyclerView()
	{
		Log.i("EventsFragment", "setRecyclerView");
		ArrayList<EventsInfo> infoList = new ArrayList<>();
		mBinding.eventsRecyclerviewEventsList.setLayoutManager(new LinearLayoutManager(getActivity()));
		mRecyclerAdapter = new EventsAdapter(infoList, R.layout.layout_events_recyclerview, mItemClickListener);
		mBinding.eventsRecyclerviewEventsList.setAdapter(mRecyclerAdapter);

        //mBinding.eventsPgb.setVisibility(View.GONE);
        //mBinding.eventsLayswipeUpdateUpdate.setVisibility(View.VISIBLE);
        //mBinding.eventsRecyclerviewEventsList.setVisibility(View.VISIBLE);

		//分割線
		DividerItemDecoration mDivider = new DividerItemDecoration(getActivity(), DividerItemDecoration.VERTICAL);
		mDivider.setDrawable(getResources().getDrawable(R.drawable.system_list_divider));
		if( mBinding.eventsRecyclerviewEventsList.getItemDecorationCount() == 0 )
		{
			mBinding.eventsRecyclerviewEventsList.addItemDecoration(mDivider);
		}
	}
```

## EventsViewModel.java

- 宣告控制ProgressBar、SwipeRefreshLayout、RecyclerView的顯示Boolean

```java=
public MutableData<Boolean> mIsShowEventsPgb = new MutableLiveData<>();
public MutableData<Boolean> mIsShowEventsLayswipe = new MutableLiveData<>();
public MutableData<Boolean> mIsShowEventsRcv = new MutableLiveData<>();
```

- 當顯示元件狀態需要更改

```java=
void showRcv(){
    mIsShowEventsLayswipe.postValue(true);
    mIsShowEventsPgb.postValue(false);
    mIsShowEventsRcv.postValue(true);
}
```

## EventsFragment.java

- 直接使用EventsViewModel.java的方法

```java=
    //設定RecyclerView的Adapter
	@SuppressLint("WrongConstant")
	private void setRecyclerView()
	{
		Log.i("EventsFragment", "setRecyclerView");
		ArrayList<EventsInfo> infoList = new ArrayList<>();
		mBinding.eventsRecyclerviewEventsList.setLayoutManager(new LinearLayoutManager(getActivity()));
		mRecyclerAdapter = new EventsAdapter(infoList, R.layout.layout_events_recyclerview, mItemClickListener);
		mBinding.eventsRecyclerviewEventsList.setAdapter(mRecyclerAdapter);

        //mBinding.eventsPgb.setVisibility(View.GONE);
        //mBinding.eventsLayswipeUpdateUpdate.setVisibility(View.VISIBLE);
        //mBinding.eventsRecyclerviewEventsList.setVisibility(View.VISIBLE);
        
        mViewModel.showRcv();
        ...
    }
```    

## fragment_events.xml

- 因為顯示是使用Boolean控制GONE/VISIBLE，所以要引入

```HTML=
<layout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:tools="http://schemas.android.com/tools">
	<data>
	    <!--引入此方法 即可控制View顯示-->
		<import type="android.view.View"/>
		
		<variable name="viewModel" type="com.example.mvvmtest2.EventsViewModel" />
	</data>
	...
</layout>	
```

- 使用方式

```html=
    <ProgressBar
			android:id="@+id/events_pgb"
			style="?android:attr/progressBarStyle"
			android:layout_width="50dp"
			android:layout_height="50dp"
			android:layout_gravity="center"
			android:paddingTop="10dp"
			android:visibility="@{viewModel.mIsShowEventsPgb ? View.VISIBLE : View.GONE}"
		/>
		<androidx.swiperefreshlayout.widget.SwipeRefreshLayout
			android:id="@+id/events_layswipe_update_update"
			android:layout_width="match_parent"
			android:layout_height="0dp"
			android:layout_weight="1"
			android:visibility="@{viewModel.mIsShowEventsLayswipe ? View.VISIBLE : View.GONE}"
		>
			<androidx.recyclerview.widget.RecyclerView
				android:id="@+id/events_recyclerview_events_list"
				android:layout_width="match_parent"
				android:layout_height="wrap_content"
				android:clipToPadding="false"
				android:scrollbars="none"
				android:visibility="@{viewModel.mIsShowEventsRcv ? View.VISIBLE : View.GONE}"
			/>
```

# 重構EventsDetailFragment.xml

## fragment_events_detail.xml

- DataBinding EventsViewModel

```html=
<layout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto">
	<data>
		<variable name="viewmodel" type="com.example.mvvmtest2.EventsViewModel" />
	</data>
	...
</layout>
```

# EventsViewModel.java

- 在EventsViewModel.java裡宣告
- 使資料做綁定後，可以直接更改資料

```java=
//EventsDetail頁面圖片
public MutableLiveData<String> mEventsDetailImg = new MutableLiveData<>();
```

- 將抓到的bundle傳入，設定給mEventDetailImg

```java=
//將輸入的url 傳入img
void setEventsClickImage(Bundle bundle){
    String url = bundle.getString(ApiDetail.Item.ITEM_URL);
    mEventDetailImg.postValue(url);
    Log.i("EventsViewModel","setClickEventsImage :" + url);
}

## fragment_events_detail.xml

- 更改ImageView

```xml=
<ImageView
    android:id="@+id/events_detail_iv_pic"
    android:layout_width="fill_parent"
    android:layout_height="match_parent"
    android:scaleType="fitXY"
    app:imageUrl="@{viewmodel.mEventDetailImg}"
/>
```

## EventsViewModel.java

- 宣告額外儲存點擊的List

```java=
//點擊儲存字串
private List<EventsInfo> mEventList;
```

- 在getEventsList()裡將data存入List

```java=
//抓取EventsList Request
void getEventsList()
	{
		mModel.sendMsgResponse(new EventsModel.itemEvents()
		{
			@Override
			public void onSuccess(List<EventsInfo> data)
			{
				//資料抓取成功
				mIsEventsDataGet.postValue(true);
				//將資料放入List
				mEventsList.postValue(data);

				mEventsClickList = data;
				Log.i("EventsViewModel", "getEventsList:onSuccess");
			}

			@Override
			public void onError(String errorResulte)
			{
				//資料抓取失敗
				mIsEventsDataGet.postValue(false);
				Log.i("EventsViewModel", "getEventsList:onError");
			}
		});
	}
```

## EventsDetailFragment.java

- 重構EventsDetailFragment.java

```java=
public class EventsDetailFragment extends Fragment
{
	@Nullable
	@Override
	public View onCreateView(
			@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState)
	{

		FragmentEventsDetailBinding mBinding = DataBindingUtil.inflate(inflater,R.layout.fragment_events_detail,container,
		                                                               false);
		View v = mBinding.getRoot();
		EventsViewModel mViewModel = new EventsViewModel();
		mBinding.setViewmodel(mViewModel);
		mBinding.setLifecycleOwner(this);

		//抓取bundle資料
		Bundle bundle = getArguments();
		mViewModel.setEventsClickImage(bundle);
		mBinding.eventsDetailIvPic.setClickable(true);

		return v;
	}
}
```