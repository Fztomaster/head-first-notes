# jQuery插件之DataTable使用
## 一.概述
### 1.什么是DataTables
Datatables是一款jquery表格插件。它是一个高度灵活的工具，可以将任何HTML表格添加高级的交互功能。
* 分页，即时搜索和排序
* 几乎支持任何数据源：DOM， javascript， Ajax 和 服务器处理
* 支持不同主题 DataTables, jQuery UI, Bootstrap, Foundation
* 各式各样的扩展: Editor, TableTools, FixedColumns ……
* 丰富多样的option和强大的API
* 支持国际化
* 免费开源 

官网网址：
http://www.datatables.club/
### 2.DataTables入门
#### 2-1.安装
在项目中使用DataTables，只需要引入三个文件即可，jQuery库，一个DT的核心js文件和一个DT的css文件。
```html
<!--第一种方式：引入Javascript / CSS （CDN）-->
<!-- DataTables CSS -->
<link rel="stylesheet" type="text/css" href="http://cdn.datatables.net/1.10.15/css/jquery.dataTables.css">
<!-- jQuery -->
<script type="text/javascript" charset="utf8" src="http://code.jquery.com/jquery-1.10.2.min.js"></script>
<!-- DataTables -->
<script type="text/javascript" charset="utf8" src="http://cdn.datatables.net/1.10.15/js/jquery.dataTables.js"></script>

<!--第二种：下载到本地-->
<!-- DataTables CSS -->
<link rel="stylesheet" type="text/css" href="DataTables-1.10.15/media/css/jquery.dataTables.css">
<!-- jQuery -->
<script type="text/javascript" charset="utf8" src="DataTables-1.10.15/media/js/jquery.js"></script>
<!-- DataTables -->
<script type="text/javascript" charset="utf8" src="DataTables-1.10.15/media/js/jquery.dataTables.js"></script>
```
#### 2-2.数据
DataTables 处理数据的三个概念：
* 处理模式
    * 客户端处理(Client )：所有的数据集预先加载（一次获取所有数据），数据处理都是在浏览器中完成的【逻辑分页】，小于10,000行数据使用 客户端处理。
    * 服务器端处理(ServerSide)—— 数据处理是在服务器上执行（页面只处理当前页的数据）【物理分页】，如果要在服务器端处理，那么需要启用serverSide属性。
* 数据类型
	* 数组(Arrays[])
	* 对象(objects{})-以对象为主，使用对象前，你需要明确告诉 DataTables 那个属性对应那一列， 通过使用columns.dataOption 或者 columns.dataOption 选项完成。
	* 实例(new myclass())
* 数据源
	* DOM
	* JavaScript
	* Ajax-主要用Ajax，需要指定请求的url
#### 2-3.选项设置

```js
$('#example').DataTable( {
    paging: false, // 通过设置paging选项，禁止表格分页(默认是打开的)
    scrollY: 400, // 设置垂直滚动条
    scrollX: 400, // 设置水平滚动条
    ajax: {
        url: "/page", // 请求的资源路径
        type："POST" // 请求方式
    }, // 异步数据源配置
    // javascript数据源配置
    // 使用对象数组，一定要配置columns，告诉 DataTables 每列对应的属性
    // data 这里是固定不变的，name，age，sex是你数据里对应的属性
    columns: [
        { title: "姓名", data: 'name' },
        { title: "年龄", data: 'age' },
        { title: "性别", data: 'sex' }
    ]
    serverSide: true, // 开启服务器模式
    columns.data： ， // 配置每一列的数据源
    searching: false, // 搜索
    ordering:  false // 排序
} );
```
后台返回的json数据格式：
```json
{
  "draw": 1,
  "recordsTotal": 500,
  "recordsFiltered": 500,
  "data": [
    {
      "name": "jack0",
      "age": "20",
      "sex": "男"
    },
    {
      "name": "rose0",
      "age": "18",
      "sex": "女"
    },
    {
      "name": "jack1",
      "age": "20",
      "sex": "男"
    },
    {
      "name": "rose1",
      "age": "18",
      "sex": "女"
    }
    ...
  ]
}
```
## 二.demo案例
### 1.后台数据准备
#### 1-1.domain层
MyPage.java
```java
/**
 * 封装数据实体
 * @author jackli
 *
 */
public class MyPage {

	private String name;
	private String age;
	private String sex;
	
	public MyPage() {
		
	}
	
	public MyPage(String name, String age, String sex) {
		super();
		this.name = name;
		this.age = age;
		this.sex = sex;
	}

	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getAge() {
		return age;
	}
	public void setAge(String age) {
		this.age = age;
	}
	public String getSex() {
		return sex;
	}
	public void setSex(String sex) {
		this.sex = sex;
	}
}
```
ResultInfo.java
```java
/**
 * 返回数据封装
 * @author jackli
 *
 */
public class ResultInfo {

	private Object data;

	public ResultInfo(Object data) {
		this.data = data;
	}

	public Object getData() {
		return data;
	}

	public void setData(Object data) {
		this.data = data;
	}
}
```
#### 1-2.web层
PageServlet.java
```java
@WebServlet(urlPatterns = "/page", name="PageServlet")
public class PageServlet extends HttpServlet {

	private static final long serialVersionUID = 1L;

	@Override
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		this.doPost(request, response);
	}

	@Override
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		List<MyPage> datas = new ArrayList<MyPage>();
		ResultInfo result = null;
		MyPage myPage = null;
		
		for (int i = 0; i < 500; i++) {
			if (i % 2 == 0) {
				myPage = new MyPage("jack" + i, "20", "男" + i);
			} else {
				myPage = new MyPage("rose" + i, "18", "女" + i);
			}
			datas.add(myPage);
		}
		
		// 解决响应内容乱码
        response.setContentType("text/html;charset=utf-8");
		
		// 返回封装数据
		result = new ResultInfo(datas);
		ObjectMapper mapper = new ObjectMapper();
		String json = mapper.writeValueAsString(result);
		response.getWriter().println(json);
	}

}
```
### 2.前端数据准备
index.jsp
```jsp
<%@ page language="java" contentType="text/html; charset=utf-8"
	pageEncoding="utf-8" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>DataTables</title>
<!-- DataTables CSS -->
<link rel="stylesheet" type="text/css" href="media/css/jquery.dataTables.css">
<!-- jQuery -->
<script type="text/javascript" src="media/js/jquery.js"></script>
<!-- DataTables -->
<script type="text/javascript" src="media/js/jquery.dataTables.js"></script>
</head>
<body>
<table id="example"  style="text-align:center" class="display">
	<!-- <thead>
		<tr>
			<td>全选/取消全选</td>
			<td>名字</td>
			<td>年龄</td>
			<td>性别</td>
			<td>操作</td>
		</tr>
	</thead>
	<tbody></tbody> -->
</table>
<script type="text/javascript">
 $(function(){
    $('#example').DataTable( {
   	  "aLengthMenu": [5, 10, 30, 60, 100],//显示页面的条数
      "paging": true, //分页
   	  "bAutoWidth": false, //自动适应表格的宽度
   	  "bProcessing": true, //进度
   	  "sScrollY": 400, //垂直的滚动条
       ajax:{
    	   url: "page" // 请求的资源
       },
    // 使用对象数组，一定要配置columns，告诉 DataTables 每列对应的属性
    // data 这里是固定不变的，name，position，salary，office 为你数据里对应的属性
    columns: [
        {title:"<input type='checkbox' class='checkboxAll'>全选/取消全选", data:null, width:"20%" }, 
        {title:"名字", data:"name", width:"20%" },
        {title:"年龄", data:"age", width:"20%"},
        {title:"性别", data:"sex", width:"20%" },
        {title:"操作", data:null, width:"20%"  }
    ],
    columnDefs: [{
    	orderable: false,//禁用排序
    	targets: [0],//指定的列
    	//返回表格的内容
    	render: function(data, type, row, meta){
    		return '<input type="checkbox" class="checkbox" name="checklist">'
    	}
      },
      {
     	targets: [4],
     	render: function(data, type, row, meta){
     		return '<button type="button">修改</button>'
     		       +'&nbsp;&nbsp;&nbsp;<button type="button">删除</button>'
     		       +'&nbsp;&nbsp;&nbsp<button type="button">详情</button>'
     	}
      }
     ],
    //国际化 
    language: {
       "sProcessing": "处理中...",
       "sLengthMenu": "显示 _MENU_ 项结果",
       "sZeroRecords": "没有匹配结果",
       "sInfo": "显示第 _START_ 至 _END_ 项结果，共 _TOTAL_ 项",
       "sInfoEmpty": "显示第 0 至 0 项结果，共 0 项",
       "sInfoFiltered": "(由 _MAX_ 项结果过滤)",
       "sInfoPostFix": "",
       "sSearch": "搜索:",
       "sUrl": "",
       "sEmptyTable": "表中数据为空",
       "sLoadingRecords": "载入中...",
       "sInfoThousands": ",",
       "oPaginate": {
           "sFirst": "首页",
           "sPrevious": "上页",
           "sNext": "下页",
           "sLast": "末页"
       },
       "oAria": {
           "sSortAscending": ": 以升序排列此列",
           "sSortDescending": ": 以降序排列此列"
       }
     }
  });
}); 

 //全选  不选择
 $(function(){
	 $(".checkboxAll").click(function() {
        if ($(this).prop("checked")) {
        	$(".checkbox").prop("checked", true);
        	$("tr").removeClass("selected");
        } else {
        	$(".checkbox").prop("checked", false);
        	$("tr").removeClass("selected");
        }		 
	 });
 });
 </script>
</body>
</html>
```