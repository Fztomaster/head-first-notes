## WebProject-travel

### 1.项目所用技术和开发模式
#### 1.1 所用技术
* 前端：HTML+css+js+bootstrap+jQuery+Ajax
* 后台：javaEE三层架构(web、service、dao)+servlet技术+filter技术
* 数据库：c3p0连接池+jdbcTemplate+mysql+redis
* 前后台数据传输：json
#### 1.2 开发模式
采用"HTML+Ajax+Servlet"的开发模式
```
前端：
	1.点击链接或打开页面
	2.通过事件触发响应行为：单击事件触发或者页面加载完成触发
	3.响应行为里发送ajax请求
		页面加载：
            $(function(){
                $.post(url, params, function(result) {
                    // 处理响应结果数据
                }, "json");
            });
        单击事件：
        	$(js对象).click(function(){
                 $.post(url, params, function(result) {
                    // 处理响应结果数据
                }, "json");
        	});
	4.从服务端得到json数据
		$.post(url, params, function(result) {
           if (result.ok) {
               // 通过jQuery的Dom来操作返回的数据
			   // 把数据显示到页面上
           } else {
               // 提示：错误信息
           }
        }, "json");
后台：
	1.web层
		定义结果ResultInfo info = null;
		try {
            1.接收参数
            2.封装实体javabean
            3.处理业务请求，调用service层
            4.结果正常的处理
            info = new ResultInfo(true, 返回的数据);
		} catch(Exception e) {
            info = new ResultInfo(false, "服务器忙，请稍等");
            e.printStackTrace();
		}
		返回结果，把ResultInfo转换成json字符串，返回给客户端
		ObjectMapper mapper = new ObjectMapper();
		String json = mapper.writeValueAsString(info);
		response.getWriter.print();
	2.service层
		1.处理业务逻辑
		2.调用dao：
			保存数据
			修改数据
			删除数据
			查询数据
		3.返回结果
	3.dao层
		操作数据库的方法
			jdbcTemplate.update()-增、删、改
			jdbcTemplate.queryForObject()-单值和单条记录
			jdbcTemplate.query()-多条记录
数据库：
	redis
	mysql
```
### 2.需求列表
* 用户模块
	* 用户注册
	* 用户激活
	* 用户登录
	* 头部显示当前登录用户的信息
	* 用户退出
	* BaseServlet
* 首页和国内游模块
	* 首页导航条-分类信息，使用Redis缓存
	* 首页黑马精选
	* 国内游列表，带分页
	* 查看旅游线路详情
* 搜索和收藏模块
	* 头部搜索线路，带分页
	* 添加收藏
	* 查看我的收藏，带分页
* 收藏排行榜模块
	* 收藏排行榜，带搜索，带分页
### 3.具体需求分析
#### 3.1 用户模块
##### 3.1.1 用户注册和用户激活
```sequence
客户端->web层:AJAX提交表单数据
Note right of web层:1.校验验证码
Note right of web层:2.接收参数map
Note right of web层:3.封装实体user(md5加密密码,设置未激活)
web层->service层:调用service层
Note right of service层:处理业务请求
service层->dao层:调用dao层
Note right of dao层:数据库操作JdbcTemplate
dao层-->service层:返回int
Note right of service层: int==1
service层-->web层:返回boolean
Note right of web层:注册成功, 发送邮箱激活
Note right of web层:响应结果封装json字符串
web层-->客户端:返回响应结果
Note right of 客户端:成功跳转登录界面
Note right of 客户端:失败弹窗提示
```
##### 3.1.2 用户登录
```sequence
客户端->web层:AJAX提交表单数据
Note right of web层:校验验证码
Note right of web层:接收请求参数
web层->service层:调用service层
service层->dao层:调用dao层
Note right of dao层:数据库操作JdbcTemplate
dao层-->service层:返回User
service层-->web层:返回User
Note right of web层:if-User!=null
Note right of web层:当前登录用户存session
Note right of web层:封装响应结果为json
web层-->客户端:返回响应结果
Note right of 客户端:登录成功跳转到首页
Note right of 客户端:登录失败提示失败信息
Note right of 客户端:重写刷新验证码
```
##### 3.1.3 头部显示当前登录用户的信息
```sequence
客户端->服务端:发送AJAX请求
Note right of 客户端:点击登录按钮
Note right of 服务端:从session域对象里取User
服务端-->客户端:当前User对象返回给客户端
Note right of 服务端:if user!=null
Note right of 客户端:if result.ok
Note right of 客户端:获取user对象，修改页面内容
Note right of 客户端:显示已登录信息，隐藏未登录信息
```
##### 3.1.4 退出
```sequence
客户端->服务端:发送AJAX请求
Note right of 客户端:点击退出按钮
Note right of 服务端:销毁当前的session对象
Note right of 服务端:request.getSession().invalidate();
Note right of 服务端:重定向到登录界面
Note right of 服务端:response.sendRedirect(request.getContextPath+"/login.html");
```
##### 3.1.5 BaseServlet
```java
public class BaseServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 先获取客户端要执行哪个方法
        String action = request.getParameter("action");
        try {
            // 获取当前Servlet对象里的action对应的方法
            Method method = this.getClass().getMethod(action, HttpServletRequest.class, HttpServletResponse.class);
            // 通过反射执行这个方法
            method.invoke(this, request, response);
        } catch(NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```
#### 3.2 首页和国内游模块
##### 3.2.1 首页导航条-分类信息，使用Redis缓存
```sequence
客户端->web层:AJAX请求
Note right of web层:1.查询所有的分类信息Categories
Note right of web层:2.再把结果放进ResultInfo里
web层->service层:处理业务请求调用service层
Note right of service层:1.先从Redis里查找数据categories
Note right of service层:2.if categories null
Note right of service层:3.调用dao，得到List<Category>
Note right of service层:4.把集合转换成json
Note right of service层:5.把json保存在Redis缓存里
Note right of service层:6.返回结果categories:String
service层->dao层:操作数据调用dao层
Note right of dao层:1.从tab_category查询所有分类信息
Note right of dao层:2.返回List<Category>
dao层-->service层:返回List<Category>
service层-->web层:返回String:categories
web层-->客户端:返回json数据
Note right of 客户端:if result.ok=true
Note right of 客户端:1.获取所有分类信息
Note right of 客户端:2.先给导航条增加"首页"
Note right of 客户端:3.再给导航条增加分类
Note right of 客户端:4.再给导航条增加"收藏排行榜"
Note right of 客户端:else alert(result.msg);
```
##### 3.2.2 首页黑马精选
```sequence
客户端->web层:AJAX请求
web层->service层:处理请求调用service
Note right of service层:1.调用dao获取人气
Note right of service层:2.调用dao获取最新
Note right of service层:3.调用dao获取主题
service层->dao层:操作数据调用dao
Note right of dao层:1.查询人气旅游数据
Note right of dao层:2.查询最新旅游数据
Note right of dao层:3.查询主题旅游数据
dao层-->service层:返回3个List<Route>
Note right of service层:1.把3类数据放进Map<String,List<Route>>
Note right of service层:2.返回map
service层-->web层:返回map
Note right of web层:把map放进ResultInfo里
web层-->客户端:返回json数据
Note right of 客户端:1.获取人气旅游数据
Note right of 客户端:2.获取最新旅游数据
Note right of 客户端:3.获取主题旅游数据
```
##### 3.2.3 国内游列表，带分页
```sequence
客户端->web层:AJAX请求
Note left of 客户端:1.点击"国内游"按钮
Note left of 客户端:2.action=queryPageByCid
Note left of 客户端:3.cid=cid值
Note left of 客户端:4.pageNumber=pageNumber值
Note left of web层:1.接收参数cid和pageNumber
Note left of web层:2.处理业务请求调用service
Note left of web层:3.queryPageByCid(cid,pageNumber)
web层->service层:调用service
Note left of service层:1.定义PageBean
Note left of service层:2.封装pageNumber
Note left of service层:3.封装总共多少条数据
Note left of service层:4.封装总共分了多少页
Note left of service层:5.页码从几开始
Note left of service层:6.页码显示到几结束
service层->dao层:调用dao
Note left of dao层:1.查询总条数
Note left of dao层:2.查询当前页数据
dao层-->service层:返回totalCount和List<Route>
service层-->web层:返回pageBean
Note right of web层:把pageBean放进ResultInfo里
web层-->客户端: 返回json数据
Note right of 客户端:1.当前页数据
Note right of 客户端:2.当前是第几页
Note right of 客户端:3.总共分了多少页
Note right of 客户端:4.总共多少条数据
```
##### 3.2.4 查看旅游线路详情
```sequence
客户端->web层:AJAX请求
Note left of 客户端:1.rid=当前线路id
Note left of 客户端:2.action=findById
Note left of web层:1.接收参数rid
Note left of web层:2.处理业务请求,调用service
web层->service层:调用service
Note left of service层:1.调用dao根据rid查询得到Route对象
Note left of service层:2.根据route的sid查询得到seller对象
Note left of service层:3.根据route的cid查询得到Category对象
Note left of service层:4.根据rid查询RouteImg集合对象
Note left of service层:5.把以上查询出的数据封装进route对象里
service层->dao层:调用dao
Note left of dao层:1.SELECT tab_route WHERE rid=?
Note left of dao层:2.SELECT * FROM tab_seller WHERE sid=?
Note left of dao层:3.SELECT * FROM tab_category WHERE cid=?
Note left of dao层:4.SELECT * FROM tab_route_img WHERE rid=?
dao层-->service层:返回Route,Seller,Category和RouteImg
service层-->web层:返回Route对象
Note right of web层:3.把Route对象放进ResultInfo
Note right of web层:4.ResultInfo转换成json数据
web层-->客户端:返回json数据
Note right of 客户端:3.显示路线信息
Note right of 客户端:4.显示商家信息
Note right of 客户端:5.显示页面顶部分类信息
Note right of 客户端:6.轮播图路径拼接
```
#### 3.3 搜索和收藏模块
##### 3.3.1 头部搜索线路，带分页
```sequence
客户端->web层:AJAX请求
Note left of 客户端:1.点击"国内游"按钮
Note left of 客户端:2.action=queryPageByCid
Note left of 客户端:3.cid=cid值
Note left of 客户端:4.pageNumber=pageNumber值
Note left of 客户端:5.rname=搜索关键字
Note left of web层:1.接收参数cid,pageNumber和rname
Note left of web层:2.处理业务请求调用service
Note left of web层:3.queryPageByCid(cid,pageNumber,rname)
web层->service层:调用service
Note left of service层:1.定义PageBean
Note left of service层:2.封装pageNumber
Note left of service层:3.封装总共多少条数据:dao.findCount(cid,rname)
Note left of service层:4.封装总共分了多少页
Note left of service层:5.封装当前页数据:dao.queryPageByCid(cid,index,rname)
Note left of service层:5.页码从几开始
Note left of service层:6.页码显示到几结束
service层->dao层:调用dao,动态拼接sql语句
Note left of dao层:1.查询总条数
Note left of dao层:2.查询当前页数据
dao层-->service层:返回totalCount和List<Route>
service层-->web层:返回pageBean
Note right of web层:4.把pageBean放进ResultInfo里
web层-->客户端: 返回json数据
Note right of 客户端:6.当前页数据
Note right of 客户端:7.当前是第几页
Note right of 客户端:8.总共分了多少页
Note right of 客户端:9.总共多少条数据
Note right of 客户端:10.分页条里每一个链接增加一个rname
```
##### 3.3.2 判断收藏状态
```sequence
客户端->web层:AJAX请求
Note left of 客户端:1.页面加载完成ajax请求服务端判断
Note left of web层:1.判断session里是否有loginUser
Note left of web层:2.判断当前用户rid是否收藏
Note left of web层:3.service.isFavorite(rid,user);
web层->service层:调用service层
Note left of service层:1.dao.findFavorite(rid,user);
service层->dao层:调用dao层
Note left of dao层:SELECT * FROM tab_favorite WHERE rid=? AND uid=?
dao层-->service层:返回Favorite对象
Note right of service层:2.favorite!=null;
service层-->web层:返回boolean
Note right of web层:4.把isFavorite放进info里
Note right of web层:5.info转换成json
web层-->客户端:返回json数据
Note right of 客户端:2.当前线路是否被当前用户收藏
Note right of 客户端:3.如果收藏了显示灰色"收藏"按钮
Note right of 客户端:4.否则显示红色"收藏"按钮
```
##### 3.3.3 添加收藏
```sequence
客户端->web层:AJAX请求
Note left of 客户端:1.点击"收藏"按钮
Note left of web层:1.判断session里是否有loginUser
Note left of web层:2.接收参数rid
Note left of web层:3.rid添加当当前用户user收藏里
Note left of web层:4.addFavorite(rid, user);
Note left of web层:5.根据rid查询route信息得到count
Note left of web层:6.未登录返回0表示
web层->service层:调用service层
Note left of service层:1.封装数据Favorite对象
Note left of service层:2.开启事务
Note left of service层:3.dao.addFavorite(template,favorite);
Note left of service层:4.dao.updateFavoriteCount(template,rid);
Note left of service层:5.提交事务
Note left of service层:6.异常回滚事务
service层->dao层:调用dao层
Note left of dao层:SELECT * FROM tab_favorite WHERE rid=? AND uid=?
dao层-->service层:返回Favorite对象
Note right of service层:2.favorite!=null;
service层-->web层:返回boolean
Note right of web层:7.把isFavorite放进info里
Note right of web层:8.info转换成json
web层-->客户端:返回json数据
Note right of 客户端:2.当前线路是否被当前用户收藏
Note right of 客户端:3.如果收藏了显示灰色"收藏"按钮
Note right of 客户端:4.否则显示红色"收藏"按钮
```
##### 3.3.4 查看我的收藏，带分页
```sequence
客户端->web层:AJAX请求
Note left of 客户端:1.打开页面myfavorite.html
Note left of 客户端:2.接收参数pageNumber
Note left of 客户端:3.页面加载完成发AJAX请求
Note left of web层:1.判断session里是否有loginUser
Note left of web层:2.获取参数pageNumber
Note left of web层:3.调用service查询指定页码的数据
Note left of web层:4.未登录返回数字0
web层->service层:调用service层
Note left of service层:1.pageBean封装数据
Note left of service层:2.设置当前页码
Note left of service层:3.每页多少条数据
Note left of service层:4.总共多少数据
Note left of service层:5.分了多少页
Note left of service层:6.页码条从几开始
Note left of service层:7.页码显示到几结束
Note left of service层:8.当前页数据集合
service层->dao层:调用dao层
Note left of dao层:1.查询总共记录数
Note left of dao层:2.查询当前页数据
dao层-->service层:返回List<Favorite>对象
service层-->web层:返回pageBean
Note right of web层:5.把pageBean放进info里
Note right of web层:6.info转换成json
web层-->客户端:返回json数据
Note right of 客户端:3.判断用户是否登录
Note right of 客户端:4.未登录跳转登录页面
Note right of 客户端:5.已登录获取当页数据并显示
Note right of 客户端:6.获取分页信息拼接分页条HTML代码并显示
```
#### 3.4 收藏排行榜模块
##### 3.4.1 收藏排行榜，带搜索，带分页 
```sequence
客户端->web层:AJAX请求
Note left of 客户端:1.action=favoriteRankRoute
Note left of 客户端:2.rname=路线名称
Note left of 客户端:3.minprice=最小金额
Note left of 客户端:4.maxprice=最大金额
Note left of 客户端:5.pageNumber=当前页码
Note left of 客户端:6.页面加载完成发AJAX请求
Note left of web层:1.接收参数rname,minprice,maxprice,pageNumber
Note left of web层:2.处理业务请求调用service
web层->service层:调用service层
Note left of service层:1.封装pageBean对象
Note left of service层:2.设置当前页码
Note left of service层:3.每页多少条数据
Note left of service层:4.总共数据的个数
Note left of service层:5.分了多少页
Note left of service层:6.页码从几开始显示
Note left of service层:7.页码显示到几结束
Note left of service层:8.当前页的数据集合
service层->dao层:调用dao层
Note left of dao层:1.查询总共的数据数
Note left of dao层:2.查询当前页的数据
dao层-->service层:返回totalCount和List<Favorite>
Note right of service层:数据全部封装成pageBean
service层-->web层:返回pageBean
Note right of web层:3.把pageBean放进info里
Note right of web层:4.info转换成json
web层-->客户端:返回json数据
Note right of 客户端:7.显示当前页的路线信息
Note right of 客户端:8.显示分页条的数据
```