>
> 在后置服务器需要对大量请求包数据进行甄别处理存储，所以常常需要对数据库操作。我们选择了sqlite3：`SQLITE是一款非常小巧的嵌入式开源数据库软件，主要具备以下几点特点`
>
>   * 1 支援大多数的SQL指令
>   * 2 一个档案就是一个数据库。不需要安装数据库服务器软件。
>   * 3 sqlite 不需要任何数据库引擎
>   * 4 完整的Unicode支援（因此没有跨语系的问题）。
>   * 5 速度很快。  
>  [更多可以参考博客](https://www.cnblogs.com/tangchao340/p/13825029.html)  
>  文章只是简单进行记录，方便自己的查询使用，如有错误，欢迎指出。
>

### 文章目录

  * 1 注意点
  * 2 函数原型
  *     * 2.1 sqlite3_open
    * 2.2 sqlite3_close
    * 2.3 sqlite3_errmsg
    * 2.4 sqlite3_exec
    * 2.5 sqlite3_get_table
    * 2.6 sqlite3_free_table
  * 3 报错宏解释
  * 4 apt-get指令 + 源码编译两种安装方式
  * 5 实例代码
  *     * 5.1 sqlite3_get_table函数使用实例
    * 5.2 sqlite3_exec函数使用实例

# 1 注意点

  * 注意1： SQLITE_OK这个宏对应的是0
  * 注意2：sqlite3_open函数中，回调函数sqlite3_callback 和它后面的void*这两个位置都可以填NULL。填NULL表示你不需要回调。比如你做insert 操作，做delete操作，就没有必要使用回调。而当你做select 时，就要使用回调，因为sqlite3 把数据查出来，得通过回调告诉你查出了什么数据。
  * 注意3：sqlite3_get_table和sqlite3_free_table配合使用，及时释放空间
  * 注意4：sqlite3_get_table第3个参数是查询结果，它依然一维数组（不要以为是二维数组，更不要以为是三维数组）。它内存布局是：字段名称，后面是紧接着是每个字段的值。
  * 注意5：有open就要有close

# 2 函数原型

## 2.1 sqlite3_open

    
    
    1. int sqlite3_open(
      const char *filename,   /* Database filename (UTF-8) */ 
      sqlite3 **ppDb          /* OUT: SQLite db handle */
    );
    功能: 比如：E:/test.db。文件名不需要一定存在，如果此文件不存在，sqlite会自动建立它。
    参数1:	filename 数据库名
    参数2：	ppDb 数据库操作句柄 (指针)
    返回值：成功返回SQLITE_OK,失败返回错误码
    

## 2.2 sqlite3_close

    
    
    2. int sqlite3_close(sqlite3* db);
    功能： 关闭一个数据库
    参数1：db操作数据库的指针(或者说句柄)
    返回值：成功返回SQLITE_OK,失败返回错误码
    

## 2.3 sqlite3_errmsg

    
    
    3. const char *sqlite3_errmsg(sqlite3* db);
    功能：通过db句柄获得操作错误信息
    返回值：返回错误信息首地址
    

## 2.4 sqlite3_exec

    
    
    4. int sqlite3_exec(
      sqlite3* db,                                  /* An open database */
      const char *sql,                           /* SQL to be evaluated */
      int (*callback)(void*arg,int,char**,char**),  /* Callback function */
      void *arg,                                    /* 1st argument to callback */
      char **errmsg                              /* Error msg written here */
    );
    
    功能：这就是执行一条sql 语句的函数。
    参数1：db 数据库操作句柄
    参数2：sql 一条sqlite语句
    参数3：callback回调函数（只有sql为查询语句的时候才执行 特别注意）
    参数4：arg 给回调函数传参，如果不需要可以填入NULL
    参数5：错误消息
    返回值：成功返回SQLITE_OK,
    
    int (*callback)(void*arg,int f_num,char** f_value,char** f_name),  /* Callback function */
    
    功能：每找到一条记录 调用一次回调函数 将结果结果输出 （从上往下一条一条查询）
    参数1：arg传递给回调函数的参数
    参数2：f_num记录中包含的字段数目（表示为列）
    参数3：f_value包含每个字段的指针数组（每列的值）
    参数4：f_name包含每个字段的名称数组 （每列的名称）
    

## 2.5 sqlite3_get_table

    
    
    5.int sqlite3_get_table(
      sqlite3 *db,          /* An open database */
      const char *zSql,     /* SQL to be evaluated */
      char ***pazResult,    /* Results of the query */
      int *pnRow,           /* Number of result rows written here */
      int *pnColumn,        /* Number of result columns written here */
      char **pzErrmsg       /* Error msg written here */
    );
    功能：不使用回调函数的查询
    参数1：db 数据表的句柄
    参数2：zsql sql的语句  
    参数3：result 指向sql语句执行结果的三级指针  此结果为申请内存存放的，它依然一维数组（不要以为是二维数组，更不要以为是三维数组）。它内存布局是：字段名称，后面是紧接着是每个字段的值。
    参数4：nrow 满足条件记录的条数，几行
    参数5：ncolumn 每条记录包含的字段数目，几列
    参数6：errsg 指向错误信息的指针
    成功返回：SQLITE_OK 失败返回错误码
    

## 2.6 sqlite3_free_table

    
    
    6.void sqlite3_free_table(char **result);
    功能：释放表的结果的内存
    参数1：指向表结果的指针
    

# 3 报错宏解释

    
    
    #define SQLITE_OK         0 /*成功结果*/
    #define SQLITE_ERROR      1 /*SQL错误或缺少数据库*/
    #define SQLITE_INTERNAL   2 /*SQLite中的内部逻辑错误*/
    #define SQLITE_PERM       3 /*访问权限被拒绝*/
    #define SQLITE_ABORT      4 /*回调例程请求中止*/
    #define SQLITE_BUSY       5 /*数据库文件已锁定*/
    #define SQLITE_LOCKED     6 /*数据库中的一个表被锁定*/
    #define SQLITE_NOMEM      7 /*一个malloc()失败*/
    #define SQLITE_READONLY   8 /*尝试写入只读数据库*/
    #define SQLITE_INTERRUPT  9 /*由sqlite3_interrupt()终止的操作*/
    #define SQLITE_IOERR      10 /*发生某种磁盘I/O错误*/
    #define SQLITE_CORRUPT    11 /*数据库磁盘映像格式不正确*/
    #define SQLITE_NOTFOUND   12 /*未知操作码sqlite3_interrupt()*/
    #define SQLITE_FULL       13 /*插入失败，因为数据库已满*/
    #define SQLITE_CANTOPEN   14 /*无法打开数据库文件*/
    #define SQLITE_PROTOCOL   15 /*数据库锁协议错误*/
    #define SQLITE_EMPTY      16 /*数据库为空*/
    #define SQLITE_SCHEMA     17 /*数据库架构已更改*/
    #define SQLITE_TOOBIG     18 /*字符串或BLOB超出大小限制*/
    #define SQLITE_CONSTRAINT 19 /*由于约束冲突而中止*/
    #define SQLITE_MISMATCH   20 /*数据类型不匹配*/
    #define SQLITE_usage      21 /*库使用错误*/
    #define SQLITE_NOLFS      22 /*使用主机不支持的操作系统功能*/
    #define SQLITE_AUTH       23 /*授权被拒绝*/
    #define SQLITE_FORMAT     24 /*辅助数据库格式错误*/
    #define SQLITE_RANGE      25 /*第二个参数定义为sqlite3_bind超出范围*/
    #define SQLITE_NOTADB     26 /*打开的不是数据库文件的文件*/
    #define SQLITE_NOTICE     27 /*来自sqlite3_log()的通知*/
    #define SQLITE_WARNING    28 /*来自sqlite3_log()的警告*/
    #define SQLITE_ROW        100 /*sqlite3_step()已准备好另一行*/
    #define SQLITE_DONE       101 /*sqlite3_step()已完成执行*/
    
    

# 4 apt-get指令 + 源码编译两种安装方式

[sqlite3两种安装方式 apt-get指令 + 源码编译
[linux]](https://blog.csdn.net/weixin_44972997/article/details/108151616?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163143099716780274177817%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163143099716780274177817&biz_id=0&utm_medium=distribute.pc_search_result.none-
task-
blog-2~all~first_rank_ecpm_v1~rank_v29_ecpm-1-108151616.pc_search_ecpm_flag&utm_term=sqlite3%20%E5%AE%89%E8%A3%85%20windalove&spm=1018.2226.3001.4187)

![在这里插入图片描述](https://img-
blog.csdnimg.cn/387a65a7e1bb4d84921b97621f2ce4fa.png?x-oss-
process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAV2luZGFsb3Zl,size_20,color_FFFFFF,t_70,g_se,x_16)

# 5 实例代码

[参考博客](https://blog.csdn.net/guanhuhousheng/article/details/6934609?utm_medium=distribute.pc_relevant.none-
task-
blog-2~default~CTRLIST~default-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-
task-blog-2~default~CTRLIST~default-2.no_search_link)

## 5.1 sqlite3_get_table函数使用实例

    
    
    int main( int , char ** )
    {
           sqlite3 * db;
           int result;
           char * errmsg = NULL;
    char **dbResult; //是 char ** 类型，两个*号
           int nRow, nColumn;
           int i , j;
    int index;
    result = sqlite3_open( “c:\\Dcg_database.db”, &db );        
               if( result != SQLITE_OK )
                 { //数据库打开失败
    return -1;
    }
    //数据库操作代码
    //假设前面已经创建了MyTable_1 表
    //开始查询，传入的dbResult 已经是 char **，这里又加了一个 & 取地址符，传递进去的就成了 char ***
    result = sqlite3_get_table( db, “select *from MyTable_1”,&dbResult, &nRow, &nColumn, &errmsg );
    if( SQLITE_OK == result )
    { //查询成功
       index = nColumn; //前面说过 dbResult 前面第一行数据是字段名称，从 nColumn 索引开始才是真正的数据
        printf( “查到%d条记录\n”, nRow );
        for(  i = 0; i < nRow ; i++ )
        {
            printf( “第 %d 条记录\n”, i+1 );
            for( j = 0 ; j < nColumn; j++ )
            {
                  printf( “字段名:%s  ?> 字段值:%s\n”,  dbResult[j], dbResult [index] );
                  ++index; // dbResult 的字段值是连续的，从第0索引到第 nColumn - 1索引都是字段名称，从第 nColumn 索引开始，后面都是字段值，它把一个二维的表（传统的行列表示法）用一个扁平的形式来表示
            }
            printf( “-------\n” );
        }
    }
    //到这里，不论数据库查询是否成功，都释放 char** 查询结果，使用 sqlite 提供的功能来释放
    sqlite3_free_table( dbResult );
    //关闭数据库
    sqlite3_close( db );
    return 0;
    }
    

## 5.2 sqlite3_exec函数使用实例

    
    
    //sqlite3的回调函数       
    // sqlite 每查到一条记录，就调用一次这个回调
    int LoadMyInfo( void * para,  intn_column,  char ** column_value,  char ** column_name )
    {
    //para是你在 sqlite3_exec 里传入的void * 参数
    //通过para参数，你可以传入一些特殊的指针（比如类指针、结构指针），然后在这里面强制转换成对应的类型（这里面是void*类型，必须强制转换成你的类型才可用）。然后操作这些数据
    //n_column是这一条记录有多少个字段 (即这条记录有多少列)
    // char ** column_value 是个关键值，查出来的数据都保存在这里，它实际上是个1维数组（不要以为是2维数组），每一个元素都是一个 char * 值，是一个字段内容（用字符串来表示，以\0结尾）
    //char ** column_name 跟column_value是对应的，表示这个字段的字段名称
    //这里，我不使用 para 参数。忽略它的存在.
    int i;
    printf( “记录包含 %d 个字段\n”, n_column );
    for( i = 0 ; i < n_column; i ++ )
    {
        printf( “字段名:%s  ?> 字段值:%s\n”,  column_name[i], column_value[i] );
    }
    printf( “------------------\n“ );       
    return 0;
    }
    int main( int , char ** )
    {
              sqlite3 * db;
              int result;
              char * errmsg =NULL;
              result =sqlite3_open( “c:\\Dcg_database.db”, &db );
                    if( result !=SQLITE_OK )
                    {
                    //数据库打开失败
    return -1;
    }
    //数据库操作代码
    //创建一个测试表，表名叫MyTable_1，有2个字段： ID 和 name。其中ID是一个自动增加的类型，以后insert时可以不去指定这个字段，它会自己从0开始增加
    result = sqlite3_exec( db, “create table MyTable_1( ID integerprimary key autoincrement, name nvarchar(32) )”, NULL, NULL, errmsg );
    if(result != SQLITE_OK )
    {
         printf( “创建表失败，错误码:%d，错误原因:%s\n”, result, errmsg );
    }
    //插入一些记录
    result = sqlite3_exec( db, “insert into MyTable_1( name ) values ( ‘走路’ )”, 0, 0, errmsg );
    if(result != SQLITE_OK )
    {
         printf( “插入记录失败，错误码:%d，错误原因:%s\n”, result, errmsg );
    }
    result = sqlite3_exec( db, “insert into MyTable_1( name ) values ( ‘骑单车’ )”, 0, 0, errmsg );
    if(result != SQLITE_OK )
    {
         printf( “插入记录失败，错误码:%d，错误原因:%s\n”, result, errmsg );
    }
    result = sqlite3_exec( db, “insert into MyTable_1( name ) values ( ‘坐汽车’ )”, 0, 0, errmsg );
    if(result != SQLITE_OK )
    {
         printf( “插入记录失败，错误码:%d，错误原因:%s\n”, result, errmsg );
    }
    //开始查询数据库
    result = sqlite3_exec( db, “select * from MyTable_1”, LoadMyInfo, NULL, errmsg );
    //关闭数据库
    sqlite3_close( db );
    return 0;
    }
    

> 如有错误，欢迎指出

