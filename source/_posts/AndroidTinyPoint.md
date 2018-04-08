---
title: 安卓学习复习笔记
date: 2015-09-08 00:02:02
categories: Android
toc: true
tags:
- Android
description: 安卓学习记录以供后期复习
---

### 进入应用程序主界面

~~~java
	/**
	 * 进入应用程序主界面
	 */
	protected void enterHome() {
		Intent intent = new Intent(this,HomeActivity.class);
		startActivity(intent);
		//在开启一个新的界面后,将导航界面关闭(导航界面只可见一次)
		finish();
	}
~~~

### 获取版本号版本名称

~~~java
/**
	 * 返回版本号
	 * @return	
	 * 			非0 则代表获取成功
	 */
	private int getVersionCode() {
		//1,包管理者对象packageManager
		PackageManager pm = getPackageManager();
		//2,从包的管理者对象中,获取指定包名的基本信息(版本名称,版本号),传0代表获取基本信息
		try {
			PackageInfo packageInfo = pm.getPackageInfo(getPackageName(), 0);
			//3,获取版本名称
			return packageInfo.versionCode;
		} catch (Exception e) {
			e.printStackTrace();
		}
		return 0;
	}

	/**
	 * 获取版本名称:清单文件中
	 * @return	应用版本名称	返回null代表异常
	 */
	private String getVersionName() {
		//1,包管理者对象packageManager
		PackageManager pm = getPackageManager();
		//2,从包的管理者对象中,获取指定包名的基本信息(版本名称,版本号),传0代表获取基本信息
		try {
			PackageInfo packageInfo = pm.getPackageInfo(getPackageName(), 0);
			//3,获取版本名称
			return packageInfo.versionName;
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}
~~~

### 系统弹窗的使用

~~~java
/**
	 * 弹出对话框,提示用户更新
	 */
	protected void showUpdateDialog() {
		//对话框,是依赖于activity存在的
		Builder builder = new AlertDialog.Builder(this);
		//设置左上角图标
		builder.setIcon(R.drawable.ic_launcher);
		builder.setTitle("版本更新");
		//设置描述内容
		builder.setMessage(mVersionDes);
		
		//积极按钮,立即更新
		builder.setPositiveButton("立即更新", new DialogInterface.OnClickListener() {
			@Override
			public void onClick(DialogInterface dialog, int which) {
				//下载apk,apk链接地址,downloadUrl
				downloadApk();
			}
		});
		
		builder.setNegativeButton("稍后再说", new DialogInterface.OnClickListener() {
			@Override
			public void onClick(DialogInterface dialog, int which) {
				//取消对话框,进入主界面
				enterHome();
			}
		});
		
		//点击取消事件监听
		builder.setOnCancelListener(new DialogInterface.OnCancelListener() {
			@Override
			public void onCancel(DialogInterface dialog) {
				//即使用户点击取消,也需要让其进入应用程序主界面
				enterHome();
				dialog.dismiss();
			}
		});
		
		builder.show();
	}
~~~

### 判断sd卡是否可用，是否挂载上及SD卡路径

~~~java
//1,判断sd卡是否可用,是否挂在上
		if(Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)){
			//2,获取sd路径
			String path = 			Environment.getExternalStorageDirectory().getAbsolutePath()
					+File.separator+"xxx.apk";
					}
~~~

### 更新安装apk代码

~~~java
	/**
	 * 安装对应apk
	 * @param file	安装文件
	 */
	protected void installApk(File file) {
		//系统应用界面,源码,安装apk入口
		Intent intent = new Intent("android.intent.action.VIEW");
		intent.addCategory("android.intent.category.DEFAULT");
		/*//文件作为数据源
		intent.setData(Uri.fromFile(file));
		//设置安装的类型
		intent.setType("application/vnd.android.package-archive");*/
		intent.setDataAndType(Uri.fromFile(file),"application/vnd.android.package-archive");
//		startActivity(intent);
		startActivityForResult(intent, 0);
	}
	
	//开启一个activity后,返回结果调用的方法 取消安装的回掉
	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		enterHome();
		super.onActivityResult(requestCode, resultCode, data);
	}
~~~

### 获取SIM卡序列号

~~~java
//获取sim卡序列号TelephoneManager
TelephonyManager manager = (TelephonyManager)
								getSystemService(Context.TELEPHONY_SERVICE);
		
String simSerialNumber = manager.getSimSerialNumber();
~~~

### 获取联系人列表

~~~java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import android.app.Activity;
import android.content.ContentResolver;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.os.Handler;
import android.text.TextUtils;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AdapterView;
import android.widget.AdapterView.OnItemClickListener;
import android.widget.BaseAdapter;
import android.widget.ListView;
import android.widget.TextView;

public class ContactListActivity extends Activity {
	protected static final String tag = "ContactListActivity";
	private ListView lv_contact;
	private List<HashMap<String,String>> contactList = new ArrayList<HashMap<String,String>>();
	private MyAdapter mAdapter;
	
	private Handler mHandler = new Handler(){
		public void handleMessage(android.os.Message msg) {
			//8,填充数据适配器
			
			mAdapter = new MyAdapter();
			lv_contact.setAdapter(mAdapter);
		};
	};

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_contact_list);
		
		initUI();
		initData();
	}
	
	class MyAdapter extends BaseAdapter{
		@Override
		public int getCount() {
			return contactList.size();
		}

		@Override
		public HashMap<String, String> getItem(int position) {
			return contactList.get(position);
		}

		@Override
		public long getItemId(int position) {
			return position;
		}

		@Override
		public View getView(int position, View convertView, ViewGroup parent) {
			View view = View.inflate(getApplicationContext(), R.layout.listview_contact_item, null);
			
			TextView tv_name = (TextView) view.findViewById(R.id.tv_name);
			TextView tv_phone = (TextView) view.findViewById(R.id.tv_phone);
			
			tv_name.setText(getItem(position).get("name"));
			tv_phone.setText(getItem(position).get("phone"));
			
			return view;
		}
		
	}

	/**
	 * 获取系统联系人数据方法
	 */
	private void initData() {
		//因为读取系统联系人,可能是一个耗时操作,放置到子线程中处理
		new Thread(){
			public void run() {
				//1,获取内容解析器对象
				ContentResolver contentResolver = getContentResolver();
				//2,做查询系统联系人数据库表过程(读取联系人权限)
				Cursor cursor = contentResolver.query(
						Uri.parse("content://com.android.contacts/raw_contacts"),
						new String[]{"contact_id"}, 
						null, null, null);
				contactList.clear();
				//3,循环游标,直到没有数据为止
				while(cursor.moveToNext()){
					String id = cursor.getString(0);
//					Log.i(tag, "id = "+id);
					//4,根据用户唯一性id值,查询data表和mimetype表生成的视图,获取data以及mimetype字段
					Cursor indexCursor = contentResolver.query(
							Uri.parse("content://com.android.contacts/data"), 
							new String[]{"data1","mimetype"}, 
							"raw_contact_id = ?", new String[]{id}, null);
					//5,循环获取每一个联系人的电话号码以及姓名,数据类型
					HashMap<String, String> hashMap = new HashMap<String, String>();
					while(indexCursor.moveToNext()){
						String data = indexCursor.getString(0);	
						String type = indexCursor.getString(1);
						
						//6,区分类型去给hashMap填充数据
						if(type.equals("vnd.android.cursor.item/phone_v2")){
							//数据非空判断
							if(!TextUtils.isEmpty(data)){
								hashMap.put("phone", data);
							}
						}else if(type.equals("vnd.android.cursor.item/name")){
							if(!TextUtils.isEmpty(data)){
								hashMap.put("name", data);
							}
						}
					}
					indexCursor.close();
					contactList.add(hashMap);
				}
				cursor.close();
				//7,消息机制,发送一个空的消息,告知主线程可以去使用子线程已经填充好的数据集合
				mHandler.sendEmptyMessage(0);
			};
		}.start();
		
	}

	private void initUI() {
		lv_contact = (ListView) findViewById(R.id.lv_contact);
		lv_contact.setOnItemClickListener(new OnItemClickListener() {
			@Override
			public void onItemClick(AdapterView<?> parent, View view,int position, long id) {
				//1,获取点中条目的索引指向集合中的对象
				if(mAdapter!=null){
					HashMap<String, String> hashMap = mAdapter.getItem(position);
					//2,获取当前条目指向集合对应的电话号码
					String phone = hashMap.get("phone");
					//3,此电话号码需要给第三个导航界面使用
					
					//4,在结束此界面回到前一个导航界面的时候,需要将数据返回过去
					Intent intent = new Intent();
					intent.putExtra("phone", phone);
					setResult(0, intent);
					
					finish();
				}
			}
		});
	}
}

~~~

### 移入移出动画

~~~java
//动画XML创建 res/anim/xxx.xml
//next_in_anim.xml
<translate
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromXDelta="100%p"
    android:toXDelta="0"
    android:duration="500">
</translate>

//next_out_anim.xml
<translate
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromXDelta="0"
    android:toXDelta="-100%p"
    android:duration="500">
</translate>

//pre_in_anim.xml
<!-- -100%p 负一屏幕的宽度大小值 -->
<translate
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromXDelta="-100%p"
    android:toXDelta="0"
    android:duration="500">
</translate>

//pre_out_anim.xml
<translate
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromXDelta="0"
    android:toXDelta="100%p"
    android:duration="500">
</translate>

overridePendingTransition(R.anim.next_in_anim, R.anim.next_out_anim);
~~~

### 图片选择器（普通、高亮）

~~~java
//selector xml 创建res/drawable/selector_xxx_xxx.xml
<selector xmlns:android="http://schemas.android.com/apk/res/android" >
    <!-- 选中深绿色图片 -->
	<item android:state_pressed="true"	android:drawable="@drawable/function_greenbutton_pressed"/>
	<!-- 浅绿色的图片 -->
	<item android:drawable="@drawable/function_greenbutton_normal"/>
</selector>
~~~

### 自定义控件属性的添加

~~~java
//attrs.xml创建  res/values/attrs.xml
<resources>
    <declare-styleable name="com.xxx.view.SettingItemView">
		<attr name="destitle" format="string"/>
		<attr name="desoff" format="string"/>
		<attr name="deson" format="string"/>
	</declare-styleable>
</resources>
~~~

### 获取可用空间的大小

~~~java


//1,获取磁盘(内存,区分于手机运行内存)可用大小,磁盘路径
		String path = Environment.getDataDirectory().getAbsolutePath();
		//2,获取sd卡可用大小,sd卡路径
		String sdPath = Environment.getExternalStorageDirectory().getAbsolutePath();
		//3,获取以上两个路径下文件夹的可用大小
		String memoryAvailSpace = Formatter.formatFileSize(this, getAvailSpace(path));
		String sdMemoryAvailSpace = Formatter.formatFileSize(this,getAvailSpace(sdPath));

//int代表多少个G	
	/**
	 * 返回值结果单位为byte = 8bit,最大结果为2147483647 bytes
	 * @param path
	 * @return	返回指定路径可用区域的byte类型值
	 */
	private long getAvailSpace(String path) {
		//获取可用磁盘大小类
		StatFs statFs = new StatFs(path);
		//获取可用区块的个数
		long count = statFs.getAvailableBlocks();
		//获取区块的大小
		long size = statFs.getBlockSize();
		//区块大小*可用区块个数 == 可用空间大小
		return count*size;
//		Integer.MAX_VALUE	代表int类型数据的最大大小
//		0x7FFFFFFF
//		
//		2147483647bytes/1024 =  2096128 KB
//		2096128KB/1024 = 2047	MB
//		2047MB = 2G
	}
~~~

### 卸载应用

~~~java
Intent intent = new Intent("android.intent.action.DELETE");
intent.addCategory("android.intent.category.DEFAULT");
intent.setData(Uri.parse("package:"+mAppInfo.getPackageName()));
startActivity(intent);
~~~

### 开启应用

~~~java
//通过桌面去启动指定包名应用
PackageManager pm = getPackageManager();
//通过Launch开启制定包名的意图,去开启应用
Intent launchIntentForPackage = pm.getLaunchIntentForPackage(mAppInfo.getPackageName());
if(launchIntentForPackage!=null){
    startActivity(launchIntentForPackage);
}else{
    ToastUtil.show(getApplicationContext(), "此应用不能被开启");
}
~~~

### 发送短信

~~~java
Intent intent = new Intent(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_TEXT,"分享一个应用,应用名称为"+mAppInfo.getName());
intent.setType("text/plain");
startActivity(intent);
~~~

### 动画集合（缩放，透明）

```java
//透明动画(透明--->不透明)
AlphaAnimation alphaAnimation = new AlphaAnimation(0, 1);
alphaAnimation.setDuration(1000);
alphaAnimation.setFillAfter(true);

//缩放动画
ScaleAnimation scaleAnimation = new ScaleAnimation(
      0, 1, 
      0, 1, 
      Animation.RELATIVE_TO_SELF, 0.5f, 
      Animation.RELATIVE_TO_SELF, 0.5f);
scaleAnimation.setDuration(1000);
alphaAnimation.setFillAfter(true);
//动画集合Set
AnimationSet animationSet = new AnimationSet(true);
//添加两个动画
animationSet.addAnimation(alphaAnimation);
animationSet.addAnimation(scaleAnimation);

//view执行动画
view.startAnimation(animationSet);
```

### ListView的使用（复用）

```java
//1,复用convertView
//2,对findViewById次数的优化,使用ViewHolder
//3,将ViewHolder定义成静态,不会去创建多个对象
//4,listView如果有多个条目的时候,可以做分页算法,每一次加载20条,逆序返回

private Handler mHandler = new Handler(){
		public void handleMessage(android.os.Message msg) {
			mAdapter = new MyAdapter();
			lv_app_list.setAdapter(mAdapter);
		};
	};
class MyAdapter extends BaseAdapter{
   
   //获取数据适配器中条目类型的总数,修改成两种(纯文本,图片+文字)
   @Override
   public int getViewTypeCount() {
      return super.getViewTypeCount()+1;
   }
   
   //指定索引指向的条目类型,条目类型状态码指定(0(复用系统),1)
   @Override
   public int getItemViewType(int position) {
      if(position == 0 || position == mCustomerList.size()+1){
         //返回0,代表纯文本条目的状态码
         return 0;
      }else{
         //返回1,代表图片+文本条目状态码
         return 1;
      }
   }
   
   //listView中添加两个描述条目
   @Override
   public int getCount() {
      return mCustomerList.size()+mSystemList.size()+2;
   }

   @Override
   public AppInfo getItem(int position) {
      if(position == 0 || position == mCustomerList.size()+1){
         return null;
      }else{
         if(position<mCustomerList.size()+1){
            return mCustomerList.get(position-1);
         }else{
            //返回系统应用对应条目的对象
            return mSystemList.get(position - mCustomerList.size()-2);
         }
      }
   }

   @Override
   public long getItemId(int position) {
      return position;
   }

   @Override
   public View getView(int position, View convertView, ViewGroup parent) {
      int type = getItemViewType(position);
      
      if(type == 0){
         //展示灰色纯文本条目
         ViewTitleHolder holder = null;
         if(convertView == null){
            convertView = View.inflate(getApplicationContext(), R.layout.listview_app_item_title, null);
            holder = new ViewTitleHolder();
            holder.tv_title = (TextView)convertView.findViewById(R.id.tv_title);
            convertView.setTag(holder);
         }else{
            holder = (ViewTitleHolder) convertView.getTag();
         }
         if(position == 0){
            holder.tv_title.setText("用户应用("+mCustomerList.size()+")");
         }else{
            holder.tv_title.setText("系统应用("+mSystemList.size()+")");
         }
         return convertView;
      }else{
         //展示图片+文字条目
         ViewHolder holder = null;
         if(convertView == null){
            convertView = View.inflate(getApplicationContext(), R.layout.listview_app_item, null);
            holder = new ViewHolder();
            holder.iv_icon = (ImageView)convertView.findViewById(R.id.iv_icon);
            holder.tv_name = (TextView)convertView.findViewById(R.id.tv_name);
            holder.tv_path = (TextView) convertView.findViewById(R.id.tv_path);
            convertView.setTag(holder);
         }else{
            holder = (ViewHolder) convertView.getTag();
         }
         holder.iv_icon.setBackgroundDrawable(getItem(position).icon);
         holder.tv_name.setText(getItem(position).name);
         if(getItem(position).isSdCard){
            holder.tv_path.setText("sd卡应用");
         }else{
            holder.tv_path.setText("手机应用");
         }
         return convertView;
      }
   }
}

static class ViewHolder{
   ImageView iv_icon;
   TextView tv_name;
   TextView tv_path;
}

static class ViewTitleHolder{
   TextView tv_title;
}

private void initList() {
		lv_app_list = (ListView) findViewById(R.id.lv_app_list);
		tv_des = (TextView) findViewById(R.id.tv_des);

  		//悬浮设置
  		lv_app_list.setOnScrollListener(new OnScrollListener() {
			@Override
			public void onScrollStateChanged(AbsListView view, int scrollState) {
				
			}
			
			@Override
			public void onScroll(AbsListView view, int firstVisibleItem,
					int visibleItemCount, int totalItemCount) {
				//滚动过程中调用方法
				//AbsListView中view就是listView对象
				//firstVisibleItem第一个可见条目索引值
				//visibleItemCount当前一个屏幕的可见条目数
				//总共条目总数
				if(mCustomerList!=null && mSystemList!=null){
					if(firstVisibleItem>=mCustomerList.size()+1){
						//滚动到了系统条目
						tv_des.setText("系统应用("+mSystemList.size()+")");
					}else{
						//滚动到了用户应用条目
						tv_des.setText("用户应用("+mCustomerList.size()+")");
					}
				}
				
			}
		});
		
}

```

###拖拽跟随移动的处理

```java
//监听某一个控件的拖拽过程(按下(1),移动(多次),抬起(1))
iv_drag.setOnTouchListener(new OnTouchListener() {
   private int startX;
   private int startY;
   //对不同的事件做不同的逻辑处理
   @Override
   public boolean onTouch(View v, MotionEvent event) {
      switch (event.getAction()) {
      case MotionEvent.ACTION_DOWN:
         startX = (int) event.getRawX();
         startY = (int) event.getRawY();
         break;
      case MotionEvent.ACTION_MOVE:
         int moveX = (int) event.getRawX();
         int moveY = (int) event.getRawY();
         
         int disX = moveX-startX;
         int disY = moveY-startY;
         
         //1,当前控件所在屏幕的(左,上)角的位置
         int left = iv_drag.getLeft()+disX;//左侧坐标
         int top = iv_drag.getTop()+disY;//顶端坐标
         int right = iv_drag.getRight()+disX;//右侧坐标
         int bottom = iv_drag.getBottom()+disY;//底部坐标
         
         //容错处理(iv_drag不能拖拽出手机屏幕)
         //左边缘不能超出屏幕
         if(left<0){
            return true;
         }
         
         //右边边缘不能超出屏幕
         if(right>mScreenWidth){
            return true;
         }
         
         //上边缘不能超出屏幕可现实区域
         if(top<0){
            return true;
         }
         
         //下边缘(屏幕的高度-22 = 底边缘显示最大值)
         if(bottom>mScreenHeight - 22){
            return true;
         }
         
         if(top>mScreenHeight/2){
            bt_bottom.setVisibility(View.INVISIBLE);
            bt_top.setVisibility(View.VISIBLE);
         }else{
            bt_bottom.setVisibility(View.VISIBLE);
            bt_top.setVisibility(View.INVISIBLE);
         }
         
         //2,告知移动的控件,按计算出来的坐标去做展示
         iv_drag.layout(left, top, right, bottom);
         
         //3,重置一次其实坐标
         startX = (int) event.getRawX();
         startY = (int) event.getRawY();
         
         break;
      case MotionEvent.ACTION_UP:
         //4,存储移动到的位置
         SpUtil.putInt(getApplicationContext(), ConstantValue.LOCATION_X, iv_drag.getLeft());
         SpUtil.putInt(getApplicationContext(), ConstantValue.LOCATION_Y, iv_drag.getTop());
         break;
      }
      //在当前的情况下返回false不响应事件,返回true才会响应事件
      
      //既要响应点击事件,又要响应拖拽过程,则此返回值结果需要修改为false
      return false;
   }
});
```

### 代码更改布局位置处理

```java
//获取屏幕宽高
mWM = (WindowManager) getSystemService(WINDOW_SERVICE);
mScreenHeight = mWM.getDefaultDisplay().getHeight();
mScreenWidth = mWM.getDefaultDisplay().getWidth();

//左上角坐标作用在iv_drag上
//iv_drag在相对布局中,所以其所在位置的规则需要由相对布局提供

//指定宽高都为WRAP_CONTENT
LayoutParams layoutParams = new RelativeLayout.LayoutParams(
      RelativeLayout.LayoutParams.WRAP_CONTENT, 
      RelativeLayout.LayoutParams.WRAP_CONTENT);
//将左上角的坐标作用在iv_drag对应规则参数上
layoutParams.leftMargin = locationX;
layoutParams.topMargin = locationY;
//将以上规则作用在iv_drag上
iv_drag.setLayoutParams(layoutParams);
```

### 获取系统联系人

```java
    /**
    * 获取系统联系人数据方法
    */
   private void initData() {
      //因为读取系统联系人,可能是一个耗时操作,放置到子线程中处理
      new Thread(){
         public void run() {
            //1,获取内容解析器对象
            ContentResolver contentResolver = getContentResolver();
            //2,做查询系统联系人数据库表过程(读取联系人权限)
            Cursor cursor = contentResolver.query(
                  Uri.parse("content://com.android.contacts/raw_contacts"),
                  new String[]{"contact_id"}, 
                  null, null, null);
            contactList.clear();
            //3,循环游标,直到没有数据为止
            while(cursor.moveToNext()){
               String id = cursor.getString(0);
//             Log.i(tag, "id = "+id);
               //4,根据用户唯一性id值,查询data表和mimetype表生成的视图,获取data以及mimetype字段
               Cursor indexCursor = contentResolver.query(
                     Uri.parse("content://com.android.contacts/data"), 
                     new String[]{"data1","mimetype"}, 
                     "raw_contact_id = ?", new String[]{id}, null);
               //5,循环获取每一个联系人的电话号码以及姓名,数据类型
               HashMap<String, String> hashMap = new HashMap<String, String>();
               while(indexCursor.moveToNext()){
                  String data = indexCursor.getString(0);    
                  String type = indexCursor.getString(1);
                  
                  //6,区分类型去给hashMap填充数据
                  if(type.equals("vnd.android.cursor.item/phone_v2")){
                     //数据非空判断
                     if(!TextUtils.isEmpty(data)){
                        hashMap.put("phone", data);
                     }
                  }else if(type.equals("vnd.android.cursor.item/name")){
                     if(!TextUtils.isEmpty(data)){
                        hashMap.put("name", data);
                     }
                  }
               }
               indexCursor.close();
               contactList.add(hashMap);
            }
            cursor.close();
            //7,消息机制,发送一个空的消息,告知主线程可以去使用子线程已经填充好的数据集合
            mHandler.sendEmptyMessage(0);
         };
      }.start();
      
   }
```

### 基类手势控制 上一页下一页

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   
   //2,创建手势管理的对象,用作管理在onTouchEvent(event)传递过来的手势动作
   gestureDetector = new GestureDetector(this, new GestureDetector.SimpleOnGestureListener(){
      @Override
      public boolean onFling(MotionEvent e1, MotionEvent e2,
            float velocityX, float velocityY) {
         //监听手势的移动
         if(e1.getX()-e2.getX()>0){
            //调用子类的下一页方法,抽象方法
            
            //在第一个界面上的时候,跳转到第二个界面
            //在第二个界面上的时候,跳转到第三个界面
            //.......
            showNextPage();
         }
         
         if(e1.getX()-e2.getX()<0){
            //调用子类的上一页方法
            //在第一个界面上的时候,无响应,空实现
            //在第二个界面上的时候,跳转到第1个界面
            //.......
            showPrePage();
         }
         
         return super.onFling(e1, e2, velocityX, velocityY);
      }
   });
}
 
//1,监听屏幕上响应的事件类型(按下(1次),移动(多次),抬起(1次))
@Override
public boolean onTouchEvent(MotionEvent event) {
   //3,通过手势处理类,接收多种类型的事件,用作处理
   gestureDetector.onTouchEvent(event);
   return super.onTouchEvent(event);
}

//下一页的抽象方法,由子类决定具体跳转到那个界面
protected abstract void showNextPage();
//上一页的抽象方法,由子类决定具体跳转到那个界面
protected abstract void showPrePage();


//点击下一页按钮的时候,根据子类的showNextPage方法做相应跳转
public void nextPage(View view){
   showNextPage();
}

//点击上一页按钮的时候,根据子类的showPrePage方法做相应跳转
public void prePage(View view){
   showPrePage();
}
```

### 手机震动

```java
//手机震动效果(vibrator 震动)
Vibrator vibrator = (Vibrator) getSystemService(Context.VIBRATOR_SERVICE);
//震动毫秒值
vibrator.vibrate(2000);
//规律震动(震动规则(不震动时间,震动时间,不震动时间,震动时间.......),重复次数)
vibrator.vibrate(new long[]{2000,5000,2000,5000}, -1);
```

### 播放声音

```java
//7,播放音乐(准备音乐,MediaPlayer)
MediaPlayer mediaPlayer = MediaPlayer.create(context, R.raw.ylzs);
mediaPlayer.setLooping(true);
mediaPlayer.start();
```

### 文本抖动效果

```java
//res/anim/shake.xml
<translate xmlns:android="http://schemas.android.com/apk/res/android" 
    android:fromXDelta="0" 
    android:toXDelta="10" 
    android:duration="1000" 
    android:interpolator="@anim/cycle_7" />
     
//cycle_7.xml
<cycleInterpolator xmlns:android="http://schemas.android.com/apk/res/android" android:cycles="7" />
   
Animation shake = AnimationUtils.loadAnimation(
        getApplicationContext(), R.anim.shake);
et_phone.startAnimation(shake);
```

### 获取手机所有应用相关信息

```java
/**
 * 返回当前手机所有的应用的相关信息(名称,包名,图标,(手机内存,sd卡),(系统,用户));
 * @param ctx  获取包管理者的上下文环境
 * @return 包含手机安装应用相关信息的集合
 */
public static List<AppInfo> getAppInfoList(Context ctx){
   //1,包的管理者对象
   PackageManager pm = ctx.getPackageManager();
   //2,获取安装在手机上应用相关信息的集合
   List<PackageInfo> packageInfoList = pm.getInstalledPackages(0);
   List<AppInfo> appInfoList = new ArrayList<AppInfo>();
   //3,循环遍历应用信息的集合
   for (PackageInfo packageInfo : packageInfoList) {
      AppInfo appInfo = new AppInfo();
      //4,获取应用的包名
      appInfo.packageName = packageInfo.packageName;
      //5,应用名称
      ApplicationInfo applicationInfo = packageInfo.applicationInfo;
      appInfo.name = applicationInfo.loadLabel(pm).toString();
      //6,获取图标
      appInfo.icon = applicationInfo.loadIcon(pm);
      //7,判断是否为系统应用(每一个手机上的应用对应的flag都不一致)
      if((applicationInfo.flags & ApplicationInfo.FLAG_SYSTEM)==ApplicationInfo.FLAG_SYSTEM){
         //系统应用
         appInfo.isSystem = true;
      }else{
         //非系统应用
         appInfo.isSystem = false;
      }
      //8,是否为sd卡中安装应用
      if((applicationInfo.flags & ApplicationInfo.FLAG_EXTERNAL_STORAGE)==ApplicationInfo.FLAG_EXTERNAL_STORAGE){
         //系统应用
         appInfo.isSdCard = true;
      }else{
         //非系统应用
         appInfo.isSdCard = false;
      }
      appInfoList.add(appInfo);
   }
   return appInfoList;
}
```

### 九宫格GridView

```java
//GridView
private void initData() {
   //准备数据(文字(9组),图片(9张))
   mTitleStrs = new String[]{
         "手机防盗","通信卫士","软件管理","进程管理","流量统计","手机杀毒","缓存清理","高级工具","设置中心"
   };
   
   mDrawableIds = new int[]{
         R.drawable.home_safe,R.drawable.home_callmsgsafe,
         R.drawable.home_apps,R.drawable.home_taskmanager,
         R.drawable.home_netmanager,R.drawable.home_trojan,
         R.drawable.home_sysoptimize,R.drawable.home_tools,R.drawable.home_settings
   };
   //九宫格控件设置数据适配器(等同ListView数据适配器)
   gv_home.setAdapter(new MyAdapter());
   //注册九宫格单个条目点击事件
   gv_home.setOnItemClickListener(new OnItemClickListener() {
      //点中列表条目索引position
      @Override
      public void onItemClick(AdapterView<?> parent, View view,int position, long id) {
         
      }
   });
}
```

### 获取空间内存（运存）大小

```java
/**
 * @param ctx  
 * @return 返回可用的内存数    bytes
 */
public static long getAvailSpace(Context ctx){
   //1,获取activityManager
   ActivityManager am = (ActivityManager) ctx.getSystemService(Context.ACTIVITY_SERVICE);
   //2,构建存储可用内存的对象
   MemoryInfo memoryInfo = new MemoryInfo();
   //3,给memoryInfo对象赋(可用内存)值
   am.getMemoryInfo(memoryInfo);
   //4,获取memoryInfo中相应可用内存大小
   return memoryInfo.availMem;
}   


/**
 * @param ctx  
 * @return 返回总的内存数 单位为bytes 返回0说明异常
 */
public static long getTotalSpace(Context ctx){
   /*//1,获取activityManager
   ActivityManager am = (ActivityManager) ctx.getSystemService(Context.ACTIVITY_SERVICE);
   //2,构建存储可用内存的对象
   MemoryInfo memoryInfo = new MemoryInfo();
   //3,给memoryInfo对象赋(可用内存)值
   am.getMemoryInfo(memoryInfo);
   //4,获取memoryInfo中相应可用内存大小
   return memoryInfo.totalMem;*/
   
   //内存大小写入文件中,读取proc/meminfo文件,读取第一行,获取数字字符,转换成bytes返回
   FileReader fileReader  = null;
   BufferedReader bufferedReader = null;
   try {
      fileReader= new FileReader("proc/meminfo");
      bufferedReader = new BufferedReader(fileReader);
      String lineOne = bufferedReader.readLine();
      //将字符串转换成字符的数组
      char[] charArray = lineOne.toCharArray();
      //循环遍历每一个字符,如果此字符的ASCII码在0到9的区域内,说明此字符有效
      StringBuffer stringBuffer = new StringBuffer();
      for (char c : charArray) {
         if(c>='0' && c<='9'){
            stringBuffer.append(c);
         }
      }
      return Long.parseLong(stringBuffer.toString())*1024;
   } catch (Exception e) {
      e.printStackTrace();
   }finally{
      try {
         if(fileReader!=null && bufferedReader!=null){
            fileReader.close();
            bufferedReader.close();
         }
      } catch (IOException e) {
         e.printStackTrace();
      }
   }
   return 0;
}  
```
### 判断服务是否开启

```java
/**
 * @param ctx  上下文环境
 * @param serviceName 判断是否正在运行的服务
 * @return true 运行 false 没有运行
 */
public static boolean isRunning(Context ctx,String serviceName){
   //1,获取activityMananger管理者对象,可以去获取当前手机正在运行的所有服务
   ActivityManager mAM = (ActivityManager) ctx.getSystemService(Context.ACTIVITY_SERVICE);
   //2,获取手机中正在运行的服务集合(多少个服务)
   List<RunningServiceInfo> runningServices = mAM.getRunningServices(1000);
   //3,遍历获取的所有的服务集合,拿到每一个服务的类的名称,和传递进来的类的名称作比对,如果一致,说明服务正在运行
   for (RunningServiceInfo runningServiceInfo : runningServices) {
      Log.i(tag, "runningServiceInfo.service.getClassName() = "+runningServiceInfo.service.getClassName());
      //4,获取每一个真正运行服务的名称
      if(serviceName.equals(runningServiceInfo.service.getClassName())){
         return true;
      }
   }
   return false;
}
```

### 多击事件

~~~java
long[] mHits = new long[3];//数组长度为点击次数
/**
 * 多次点击
 * 
 * @param view
 */
public void onClick(View view) {
	//		src 源数组
	//		srcPos 开始拷贝的位置
	//		dst 目标数组
	//		dstPos 目标数组的起始拷贝位置
	//		length 拷贝的数组长度
	System.arraycopy(mHits, 1, mHits, 0, mHits.length - 1);//拷贝数组
	mHits[mHits.length - 1] = SystemClock.uptimeMillis();
	if (mHits[0] >= (SystemClock.uptimeMillis() - 500)) {
		System.out.println("是男人!!!");
		mHits = new long[3];
	}
}
~~~

