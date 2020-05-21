## Homework6

- 下载app-debug.apk

- 对应目录下终端运行

  **adb install -t app-debug.apk**

### TODO 定义数据库名、版本；创建数据库

```java
// TODO 定义数据库名、版本；创建数据库
public static final int DATABASE_VERSION = 1;
public static final String DATABASE_NAME = "Todo.db";

public TodoDbHelper(Context context) {
    super(context, DATABASE_NAME, null, DATABASE_VERSION);
}

@Override
public void onCreate(SQLiteDatabase db) {
    db.execSQL(SQL_CREATE_ENTRIES);
}

@Override
public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
    db.execSQL(SQL_DELETE_ENTRIES);
    onCreate(db);
}
```

### TODO 定义表结构和 SQL 语句常量

```java
// TODO 定义表结构和 SQL 语句常量
public static final String SQL_CREATE_ENTRIES =
        "CREATE TABLE " + TodoEntry.TABLE_NAME + " (" +
                TodoEntry._ID + " INTEGER PRIMARY KEY," +
                TodoEntry.COLUMN_NAME_CONTENT + " TEXT," +
                TodoEntry.COLUMN_NAME_STATE +" TEXT," +
                TodoEntry.COLUMN_NAME_DATE + " TEXT)";

public static final String SQL_DELETE_ENTRIES = "DROP TABLE IF EXISTS " + TodoEntry.TABLE_NAME;

private TodoContract() {
}

public static class TodoEntry implements BaseColumns {

    public static final String TABLE_NAME = "todo_homework";

    public static final String COLUMN_NAME_CONTENT = "content";

    public static final String COLUMN_NAME_DATE = "date";

    public static final String COLUMN_NAME_STATE = "state";
}
```

### TODO 查询功能

```java
   private List<Note> loadNotesFromDatabase() throws ParseException {
        // TODO 从数据库中查询数据，并转换成 JavaBeans
        List<Note> notes= new ArrayList<>();
        SQLiteDatabase db = dbHelper.getReadableDatabase();
        String[] projection = {
                BaseColumns._ID,
                TodoContract.TodoEntry.COLUMN_NAME_CONTENT,
                TodoContract.TodoEntry.COLUMN_NAME_STATE,
                TodoContract.TodoEntry.COLUMN_NAME_DATE
        };

        // How you want the results sorted in the resulting Cursor
        String sortOrder =
                TodoContract.TodoEntry.COLUMN_NAME_DATE + " DESC";

        Cursor cursor = db.query(
                TodoContract.TodoEntry.TABLE_NAME,   // The table to query
                projection,             // The array of columns to return (pass null to get all)
                null,              // The columns for the WHERE clause
                null,          // The values for the WHERE clause
                null,                   // don't group the rows
                null,                   // don't filter by row groups
                sortOrder               // The sort order
        );

        Log.i(TAG, "perfrom query data:");
        while (cursor.moveToNext()) {
            long itemId = cursor.getLong(cursor.getColumnIndexOrThrow(TodoContract.TodoEntry._ID));
            String content = cursor.getString(cursor.getColumnIndex(TodoContract.TodoEntry.COLUMN_NAME_CONTENT));
            int state = Integer.parseInt(cursor.getString(cursor.getColumnIndex(TodoContract.TodoEntry.COLUMN_NAME_STATE)));
            String date = cursor.getString(cursor.getColumnIndex(TodoContract.TodoEntry.COLUMN_NAME_DATE));
            Log.i(TAG, "itemId:" + itemId + ", content:" + content + ", date:" + date);
            Note note = new Note(itemId);
            if(state == 1)
                note.setState(State.DONE);
            else
                note.setState(State.TODO);
            note.setContent(content);
            note.setDate(SIMPLE_DATE_FORMAT.parse(date));
            notes.add(note);
        }
        cursor.close();
        return notes;
    }
```

### TODO 删除数据

```java
private void deleteNote(Note note) {
    // TODO 删除数据
    SQLiteDatabase db = dbHelper.getWritableDatabase();

    String selection = TodoContract.TodoEntry._ID + " LIKE ?";
    // Specify arguments in placeholder order.
    String[] selectionArgs = {String.valueOf(note.id)};
    // Issue SQL statement.
    int deletedRows = db.delete(TodoContract.TodoEntry.TABLE_NAME, selection, selectionArgs);
    Log.i(TAG, "perform delete data, result:" + deletedRows);
}
```

### TODO 更新数据

```java
    private void updateNode(Note note) {
        // 更新数据
        SQLiteDatabase db = dbHelper.getWritableDatabase();
        // New value for one column
        ContentValues values = new ContentValues();
        int STATE;
        if(note.getState().equals(State.TODO))
            STATE = 0;
        else
            STATE = 1;

        values.put(TodoContract.TodoEntry.COLUMN_NAME_STATE, String.valueOf(STATE));
        // Which row to update, based on the title
        String selection = TodoContract.TodoEntry._ID + " LIKE ?";

        String[] selectionArgs = {String.valueOf(note.id)};


        int count = db.update(
                TodoContract.TodoEntry.TABLE_NAME,
                values,
                selection,
                selectionArgs);
        Log.i(TAG, "perform update data, result:" + count);
    }
}
```

### TODO 插入数据

```java
private boolean saveNote2Database(String content) {
    // TODO 插入一条新数据，返回是否插入成功
    SQLiteDatabase db = dbHelper.getWritableDatabase();

    // Create a new map of values, where column names are the keys
    ContentValues values = new ContentValues();
    values.put(TodoContract.TodoEntry.COLUMN_NAME_CONTENT, content);
    //获取当前时间
    Date date = new Date(System.currentTimeMillis());
    values.put(TodoContract.TodoEntry.COLUMN_NAME_DATE, SIMPLE_DATE_FORMAT.format(date));
    values.put(TodoContract.TodoEntry.COLUMN_NAME_STATE, "0");

    // Insert the new row, returning the primary key value of the new row
    long newRowId = db.insert(TodoContract.TodoEntry.TABLE_NAME, null, values);
    Log.i(TAG, "perform add data, result:" + newRowId);
    if(newRowId > 0)
        return true;
    else
        return false;
}
```