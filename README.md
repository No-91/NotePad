# NotePad
一.实验

基于NotePad应用做功能扩展

二.实验内容

1、基本要求：

(1)NoteList中显示条目增加时间戳显示

(2)添加笔记查询功能（根据标题查询）

2、附加功能

(1)UI美化

更改主题颜色，美化UI界面；

(2）背景更换

更改笔记背景颜色；

(3)

文本字体大小及颜色改变


三.功能截图以及部分代码展示

💚功能一实现：NoteList中显示条目增加时间戳显示
1.在notelist_item.xml布局中，添加Textview显示时间戳
```
<TextView
       android:id="@+id/text2"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:textAppearance="?android:attr/textAppearanceLarge"
       android:textSize="12dp"
       android:gravity="center_vertical"
       android:paddingLeft="10dip"
       android:singleLine="true"
       android:layout_weight="1"
       android:layout_margin="0dp"
       />
```
2.在NoteEditor中添加SimpleDateFormat调整时间类型
```
Date nowTime = new Date(System.currentTimeMillis());
               SimpleDateFormat sdFormatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String retStrFormatNowDate = sdFormatter.format(nowTime);
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, retStrFormatNowDate);
```
3.运行显示效果：


![image](https://github.com/No-91/NotePad/blob/master/images/111.png)

💚功能二实现：添加笔记查询功能（根据标题查询）

1.在NoteList.java中添加searchview实现查询功能

```
    private void SearchView(){
        searchView=findViewById(R.id.sv);
        searchView.onActionViewExpanded();
        searchView.setQueryHint("输入搜索笔记内容");
        searchView.setSubmitButtonEnabled(true);
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String s) {
                return false;
            }

            @Override
            public boolean onQueryTextChange(String s) {
                if(!s.equals("")){
                    String selection=NotePad.Notes.COLUMN_NAME_TITLE+" GLOB '*"+s+"*'";
                    updatecursor = getContentResolver().query(
                            getIntent().getData(),            // 使用提供程序的默认内容URI。
                            PROJECTION,                       // 返回每个便笺的便笺ID和标题。
                            selection,                        // 没有WHERE子句，返回所有记录。
                            null,                             
                            NotePad.Notes.DEFAULT_SORT_ORDER  // 使用默认排序顺序。
                    );
                    if(updatecursor.moveToNext())
                        Log.i("daawdwad",selection);
                }
                else {
                    updatecursor = getContentResolver().query(
                            getIntent().getData(),            // 使用提供程序的默认内容URI。
                            PROJECTION,                       // 返回每个便笺的便笺ID和标题。
                            null,                           
                            null,                             // 没有WHERE子句，因此没有WHERE列值。
                            NotePad.Notes.DEFAULT_SORT_ORDER  // 使用默认排序顺序。
                    );
                }
                adapter.swapCursor(updatecursor);
                return false;
            }
        });
    }
```
2.运行显示效果：

![iamge](https://github.com/No-91/NotePad/blob/master/images/222.png)


💚功能三实现：UI美化--更改主题颜色，美化UI界面；

1.自定义一个MyCursorAdapter类继承SimpleCursorAdapter，完成cursor读取的数据库内容填充到item，将颜色填充：

```
public class MyCursorAdapter extends SimpleCursorAdapter {
    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            case NotePad.Notes.PINK_COLOR:
                view.setBackgroundColor(Color.rgb(255, 192, 203));
                break;
            case NotePad.Notes.VIOLET_COLOR:
                view.setBackgroundColor(Color.rgb(221, 160, 221));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
}
```

2.在NoteColor.java文件处理更换背景颜色操作：
```
public class NoteColor extends Activity {
    private Cursor mCursor;
    private Uri mUri;
    private int color;
    private static final int COLUMN_INDEX_TITLE = 1;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_color);
        //从NoteEditor传入的uri
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
    }
    @Override
    protected void onResume(){
        //执行顺序在onCreate之后
        if (mCursor != null) {
            mCursor.moveToFirst();
            color = mCursor.getInt(COLUMN_INDEX_TITLE);
        }
        super.onResume();
    }
    @Override
    protected void onPause() {
        //执行顺序在finish()之后，将选择的颜色存入数据库
        super.onPause();
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
        getContentResolver().update(mUri, values, null, null);
    }
    public void white(View view){
        color = NotePad.Notes.DEFAULT_COLOR;
        finish();
    }
    public void yellow(View view){
        color = NotePad.Notes.YELLOW_COLOR;
        finish();
    }
    public void blue(View view){
        color = NotePad.Notes.BLUE_COLOR;
        finish();
    }
    public void green(View view){
        color = NotePad.Notes.GREEN_COLOR;
        finish();
    }
    public void red(View view){
        color = NotePad.Notes.RED_COLOR;
        finish();
    }
    public void pink(View view){
        color = NotePad.Notes.PINK_COLOR;
        finish();
    }
    public void violet(View view){
        color = NotePad.Notes.VIOLET_COLOR;
        finish();
    }

}

```
3.在note_color.xml布局放置7个ImageButton文件实现更改背景颜色:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorWhite"
        android:onClick="white"/>
    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorYellow"
        android:onClick="yellow"/>
    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorBlue"
        android:onClick="blue"/>
    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorGreen"
        android:onClick="green"/>
    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorRed"
        android:onClick="red"/>
    <ImageButton
        android:id="@+id/color_pink"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorPink"
        android:onClick="pink"/>
    <ImageButton
        android:id="@+id/color_violet"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorViolet"
        android:onClick="violet"/>
</LinearLayout>
```
4.NotePad中预定义好背景色，每一种颜色对应不同的int值
```
 public static final String COLUMN_NAME_MODIFICATION_DATE = "modified";
        public static final String COLUMN_NAME_BACK_COLOR = "color";
        public static final int DEFAULT_COLOR = 0; //白色
        public static final int YELLOW_COLOR = 1; //黄色
        public static final int BLUE_COLOR = 2; //蓝色
        public static final int GREEN_COLOR = 3; //绿色
        public static final int RED_COLOR = 4; //红色
        public static final int PINK_COLOR = 5;//粉色
        public static final int VIOLET_COLOR=6;// 浅紫色
```

5.在NotePadProvider表的地方添加颜色字段：

```
 @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                    + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                    + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                    + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                    + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                    + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " Text,"
                    + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER"//color
                    + ");");
        }
```

6.运行显示截图：

更改之前默认为白色：

![iamge](https://github.com/No-91/NotePad/blob/master/images/333.png)

更改为蓝色：

![iamge](https://github.com/No-91/NotePad/blob/master/images/444.png)

💚功能三实现：更改记事本的背景颜色：

在editor_options_menu.xml中添加一个更改背景的选项：

```
<item android:id="@+id/menu_color"
        android:title="@string/menu_color"
        android:icon="@drawable/change_background_color"
        app:showAsAction="always" >
    </item>

```
在NoteEditor中添加changecolor()，在显示的7种颜色中选择一种背景颜色：

默认背景为白色：


选择颜色：


更改后：



