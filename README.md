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
