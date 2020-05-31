# 1.第一项功能
## 1.1更改布局
因为要增加时间戳，所以一个控件是不够的，notelist_item.xml是要改的。需要增加一个TextView用于显示时间戳。
这里采用线性布局，将笔记标题和时间戳做垂直（vertical）的线性布局
![图片描述]()
### 整体代码如下：
```bash
<?xml version="1.0" encoding="utf-8"?>

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">


    <TextView
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
        android:textAppearance="?android:attr/textAppearanceLarge" />

    <TextView
        android:id="@android:id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        />


</LinearLayout>
```
### 布局截图
![图片描述]()



## 1.2在contentProvider类里面增加对时间读取
时间的记录系统本来就有，所以只需要将其读取出来就可以了

```java
    private static final String[] READ_NOTE_PROJECTION = new String[] {
            NotePad.Notes._ID,               // Projection position 0, the note's id
            NotePad.Notes.COLUMN_NAME_NOTE,  // Projection position 1, the note's content
            NotePad.Notes.COLUMN_NAME_TITLE, // Projection position 2, the note's title

            //增加读取的数据
            //NotePad.Notes.COLUMN_NAME_CREATE_DATE,    //想增加 显示创建时间 时可用
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
    };
```



##  1.3显示读取到的时间
在OnCreate()方法上增加显示

```java
        //增加
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;

        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        
        //加上包含时间戳的组件的id

        int[] viewIDs = { android.R.id.text1 ,android.R.id.text2 };
```


##  1.4格式化时间
老师说系统提示的时间是根据毫秒算的
，所以显示的时候还需要一个将数值格式化为时间的方法

不然时间就会显示形成一串数字：

![图片描述]()
代码更改如下：
```java
        ContentValues values = new ContentValues();

        SimpleDateFormat dateformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String dateStr = dateformat.format(System.currentTimeMillis());
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateStr);

```


## 1.5最终结果
![图片描述]()


# 2第二项功能
参考博客：[https://www.cnblogs.com/liuleliu/p/12256918.html](https://www.cnblogs.com/liuleliu/p/12256918.html)
## 2.1更改布局
在菜单栏增加一个item用于搜索，
也就是在menu文件夹下增加一个文件
并附上一个对应意思的小图标
![图片描述]()
布局代码

```bash
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/search"
        android:icon="@drawable/search"
        android:title="Search"
        android:actionViewClass="android.widget.SearchView"
        android:showAsAction="always"
    ></item>


    </menu>
```
布局效果图片
![图片描述]()

## 2.2编写事件


在notelist的onCreateOptionsMenu()方法里面增加对 搜索item 的响应

代码如下

```java
        getMenuInflater().inflate(R.menu.search_menu,menu);
        MenuItem search=menu.findItem(R.id.search);
        SearchView mysearchview=(SearchView)search.getActionView();
        mysearchview.setQueryHint("搜索笔记");
        mysearchview.setOnQueryTextListener(new SearchView.OnQueryTextListener(){
            @Override
//当提交搜索框内容后执行的方法
            public boolean onQueryTextSubmit(String query) {
                return false;
            }

            @Override
//当搜索框内内容改变时执行的方法
            public boolean onQueryTextChange(String newText) {
                refresh(newText);//数据更新函数，newText为获取到的搜索框中内容
                return false;
            }
        });
```

点击 放大镜小图标 之后响应如下

![图片描述]()

### 2.2.1搜索响应事件
更新SQL查询语句，刷新游标，把数据传入适配器

```java
    void refresh(String key)
    {

        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
        int[] viewIDs = { android.R.id.text1 ,android.R.id.text2 };
        //添加游标搜索条件
        String  setSelection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = { "%"+key +"%"};
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note.
                setSelection ,                             // No where clause, return all records.
                selectionArgs,                             // No where clause, therefore no where column values.
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );

        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,          // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
    }


```

## 2.3 结果
### 笔记截图
![图片描述]()

### 搜索结果截图
![图片描述]()
