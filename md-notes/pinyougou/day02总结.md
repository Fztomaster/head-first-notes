# day02总结

## 一.AngularJs

### 1.四大特性

- **mvc开发模式**
  - model：数据模型，**ng-model**
  - view：视图，表达式展示视图数据，**{{变量或表达式}}**
  - controller：控制器
- **数据双向绑定**
  - 模型数据发送变化的同时，视图数据随之变化
  - 视图数据发生变化的同时，模型数据随之变化
- **依赖注入**
  - 解耦合、代码复用性更强
- **模块化开发(实现程序的高内聚、低耦合)**
  - 高内聚：模块内部完成的功能高度集中
  - 低耦合：模块之间互相不影响

### 2.angularJs基础语法

#### 2-1.表达式

- **ng-app**：定义angular模块和angular的作用范围
- **{{}}**：里面放表达式

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>angular-demo1 表达式</title>
    <!--引入angularJS-->
    <script type="text/javascript" src="../plugins/angularjs/angular.min.js">
    </script>
</head>
<!--ng-app 定义angularJs模块 定义angularJs作用范围-->
<body ng-app>
<!--{{表达式}}，表达式可以是变量或者运算式-->
    {{100+100}}
</body>
</html>
```

#### 2-2.数据双向绑定

- **ng-model**：用于绑定数据，等价于jQuery的$("#id").val

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>angular-demo2 数据双向绑定</title>
    <!--引入angularJS-->
    <script type="text/javascript" src="../plugins/angularjs/angular.min.js">
    </script>
</head>
<!--ng-app 定义angularJs模块 定义angularJs作用范围-->
<body ng-app>
    <!--ng-model绑定数据模型model，相当于${"#input"}.val-->
    请输入：<input ng-model="name">
    <!--通过view视图展示绑定的模型变量中的数据-->
    输出的内容：{{ name }}
</body>
</html>
```

#### 2-3.控制器

- **ng-app="myApp"**
  - 定义angular模块
- **ng-controller="myCtrl"**
  - 定义控制器
- **ng-init="name='肉丝'"**
  - 初始化name的值
- **var app = angular.module("myApp", []);** 
  - 初始化模块
  - 参数一：模块名称
  - 参数二：依赖的其他模块，如果没有依赖，也需要定义[]
- **app.controller("myCtrl", function($scope) {....})**
  - 初始化控制器
  - 参数一：控制器名称
  - $scope：全局作用域对象，作用：模型数据与视图数据交互桥梁

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>angular-demo3 控制器</title>
    <!--引入angularJS-->
    <script type="text/javascript" src="../plugins/angularjs/angular.min.js"></script>
    <script type="text/javascript">
        // 初始化模块  参数一：模块名称  参数二：依赖的其他模块，如果没有依赖，也需要定义[]
        var app = angular.module("myApp", []);
        // 初始化（定义）控制器  参数一：控制器名称   $scope：全局作用域对象，作用：模型数据与视图数据交互的桥梁。
        // $scope内置对象
        app.controller("myCtrl", function ($scope) {
            // 封装模型数据
            // $scope.name = "杰克";
        });
    </script>
</head>
<!--ng-app 定义angularJs模块 定义angularJs作用范围-->
<!--ng-init初始化name的值-->
<body ng-app="myApp" ng-controller="myCtrl" ng-init="name='肉丝'">
    <!--ng-model绑定数据模型model，相当于${"#input"}.val-->
    请输入：<input ng-model="name">
    <!--通过view视图展示绑定的模型变量中的数据-->
    输出的内容：{{ name }}
</body>
</html>
```

#### 2-4.事件指令

- **ng-click="add()"**
  - 绑定事件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>angular-demo4 事件指令</title>
    <!--引入angularJS-->
    <script type="text/javascript" src="../plugins/angularjs/angular.min.js"></script>
    <script type="text/javascript">
        // 初始化模块  参数一：模块名称  参数二：依赖的其他模块，如果没有依赖，也需要定义[]
        var app = angular.module("myApp", []);
        // 初始化（定义）控制器  参数一：控制器名称   $scope：全局作用域对象，作用：模型数据与视图数据交互的桥梁。
        // $scope内置对象
        app.controller("myCtrl", function ($scope) {
            $scope.add = function () {
                $scope.z = parseInt($scope.x) + parseInt($scope.y);
            }
        });
    </script>
</head>
<!--ng-app 定义angularJs模块 定义angularJs作用范围-->
<body ng-app="myApp" ng-controller="myCtrl">
    x：<input ng-model="x">
    y: <input ng-model="y">
    <button ng-click="add()">相加</button>
    相加结果：{{z}}
</body>
</html>
```

#### 2-5.循环数组

- **ng-repeat="x in list"**
  - 循环遍历数组
  - 变量 in 数组值
- **$scope.list=['a', 'b', ...]**
  - 定义模型数据

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>angular-demo5 循环数组</title>
    <!--引入angularJS-->
    <script type="text/javascript" src="../plugins/angularjs/angular.min.js"></script>
    <script type="text/javascript">
        // 初始化模块  参数一：模块名称  参数二：依赖的其他模块，如果没有依赖，也需要定义[]
        var app = angular.module("myApp", []);
        // 初始化（定义）控制器  参数一：控制器名称   $scope：全局作用域对象，作用：模型数据与视图数据交互的桥梁。
        // $scope内置对象
        app.controller("myCtrl", function ($scope) {
            // 封装数据模型
            $scope.list = ["jack", "rose", "tom"];
        });
    </script>
</head>
<!--ng-app 定义angularJs模块 定义angularJs作用范围-->
<body ng-app="myApp" ng-controller="myCtrl">
   <table>
       <!--ng-repeat循环遍历数组  变量  in 数组值-->
       <tr ng-repeat="x in list">
           <td>{{x}}</td>
       </tr>
   </table>
</body>
</html>
```

#### 2-6.循环对象数组

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>angular-demo6 循环对象数组</title>
    <!--引入angularJS-->
    <script type="text/javascript" src="../plugins/angularjs/angular.min.js"></script>
    <script type="text/javascript">
        // 初始化模块  参数一：模块名称  参数二：依赖的其他模块，如果没有依赖，也需要定义[]
        var app = angular.module("myApp", []);
        // 初始化（定义）控制器  参数一：控制器名称   $scope：全局作用域对象，作用：模型数据与视图数据交互的桥梁。
        // $scope内置对象
        app.controller("myCtrl", function ($scope) {
            // 封装数据模型
            $scope.userList = [
                {
                    "id": 1,
                    "name": "jack"
                },
                {
                    "id": 2,
                    "name": "rose"
                },
                {
                    "id": 3,
                    "name": "tom"
                }
            ];
        });
    </script>
</head>
<!--ng-app 定义angularJs模块 定义angularJs作用范围-->
<body ng-app="myApp" ng-controller="myCtrl">
   <table>
       <tr>
           <th>编号</th>
           <th>姓名</th>
       </tr>
       <!--ng-repeat循环遍历数组  变量  in 数组值-->
       <tr ng-repeat="user in userList">
           <td>{{user.id}}</td>
           <td>{{user.name}}</td>
       </tr>
   </table>
</body>
</html>
```

#### 2-7.内置服务

- **$scope**
  - 全局作用域对象
  - 作用：模型数据与视图数据交互的桥梁
- **$http**
  - 发送http请求的内置对象
  - **$http.get(url).success(function(response) {.....})**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>angular-demo7 内置服务</title>
    <!--引入angularJS-->
    <script type="text/javascript" src="../plugins/angularjs/angular.min.js"></script>
    <script type="text/javascript">
        // 初始化模块  参数一：模块名称  参数二：依赖的其他模块，如果没有依赖，也需要定义[]
        var app = angular.module("myApp", []);
        // 初始化（定义）控制器  参数一：控制器名称   $scope：全局作用域对象，作用：模型数据与视图数据交互的桥梁。
        // $scope内置对象，$http发请求，也是内置对象
        app.controller("myCtrl", function ($scope, $http) {
            $http.get("../brand/findAll.do").success(function (response) {
                $scope.brandList = response;
            });
        });
    </script>
</head>
<!--ng-app 定义angularJs模块 定义angularJs作用范围-->
<body ng-app="myApp" ng-controller="myCtrl">
   <table>
       <tr>
           <th>编号</th>
           <th>名称</th>
           <th>首字母</th>
       </tr>
       <!--ng-repeat循环遍历数组  变量  in 数组值-->
       <tr ng-repeat="brand in brandList">
           <td>{{brand.id}}</td>
           <td>{{brand.name}}</td>
           <td>{{brand.firstChar}}</td>
       </tr>
   </table>
</body>
</html>
```

## 二.需求

### 1.品牌列表

#### 1-1.思路

- 前端用angularJs的$http请求
- 后台编写查询所有商标的代码

#### 1-1.引入angularJs

在brand.html中

```html
<!--引入angularJs-->
<script src="../plugins/angularjs/angular.min.js"></script>
```



### 2.品牌列表分页

### 3.增加品牌

### 4.修改品牌

### 5.删除品牌

### 条件查询品牌