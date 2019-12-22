# PageHelper使用

## yml文件配置

```yml
  pagehelper:
      helperDialect: mysql
    #  分页合理化，输入值小于1 返回第一页数据，大于总页数返回最后一页数据
      reasonable: true
      supportMethodsArguments: true
      params: count=countSql
```

## 代码

```java
/**
     * 获得用户，分页
     * @param currPage
     * @return
     */
    @RequestMapping("/getUsers")
    @ResponseBody
    public PageInfo<User> getUsers(@RequestParam(value = "currentPage", defaultValue = "1") int currPage) {
        //推荐结果个数
        int PAGE_SIZE = 20;
//        return userService.queryUsersBySql(currPage, PAGE_SIZE);
        //初始化分页插件
        //填入当前页和每页显示数量
        PageHelper.startPage(currPage,PAGE_SIZE);
        //查询所有数据
        List<User> users = userService.getAllUsers();
        // 使用 pageinfo 实现分页
        PageInfo<User> pageInfo = new PageInfo<>(users);
        return pageInfo;
    }
```

## pageinfo参数详解

```java
 	//当前页
    private int pageNum;
 
    //每页的数量
    private int pageSize;
 
    //当前页的数量
    private int size;
 
    //由于startRow和endRow不常用，这里说个具体的用法
    //可以在页面中"显示startRow到endRow 共size条数据"
    //当前页面第一个元素在数据库中的行号
    private int startRow;
 
    //当前页面最后一个元素在数据库中的行号
    private int endRow;
    //总记录数
    private long total;
 
    //总页数
    private int pages;
 
    //结果集(每页显示的数据)
    private List<T> list;
 
    //第一页
    private int firstPage;
 
    //前一页
    private int prePage;
 
    //是否为第一页
    private boolean isFirstPage = false;
 
    //是否为最后一页
    private boolean isLastPage = false;
 
    //是否有前一页
    private boolean hasPreviousPage = false;
 
    //是否有下一页
    private boolean hasNextPage = false;
 
    //导航页码数
    private int navigatePages;
 
    //所有导航页号
    private int[] navigatepageNums;
```

