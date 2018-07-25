[![Build Status](https://travis-ci.org/BrooksYang/laravel-api-helper.svg?branch=master)](https://travis-ci.org/BrooksYang/laravel-api-helper)
[![Software License][ico-license]](LICENSE)

[ico-license]: https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat

## 介绍
该项目是基于Laravel 5.5的API文档生成工具，根据laravel路由及注释文件生成可视化api文档，并采用guzzlehttp进行调试。

## Demo
[点击访问](http://api-helper.brooksyang.cn/api/docs)

## 项目依赖
|   依赖包    |   版本  |
|   :---     | :----: |
|    PHP     |  >=7.0 |
|   Laravel  |  >=5.5 |
|   predis   |  >=1.1 |
| Guzzlehttp |  >=6.3 |

## 更新日志

```php
v1.4.3 模块分组优化

v1.4.2 新增模块分组功能，以区分不同命名空间下的重名文件夹

v1.4.1 修复api-helper配置文件的bug

v1.4.0 可指定文档生成的命名空间，向下兼容v1.3，默认命名空间为App\Http\Controllers

v1.3.2 压力测试安全机制优化

v1.3.0 表单支持文件上传

v1.2.0 新增ResponseHelper Trait，封装常用api返回方法

v1.1.1 Guzzle 请求异常处理

v1.1.0 支持在线ApacheBench服务器压力测试，并缓存最近一次压测结果，在接口列表及详情页展示
```

## 安装
```php
composer require brooksyang/laravel-api-helper
```

## 配置
打印配置文件，及css资源，该命令将生成config/api-helper.php，及/public/vendor/api_doc/css/bulma.css文件
```php
php artisan vendor:publish --tag=api-doc
```

api-helper.php 配置项说明
```php
cache_tag_prefix // 缓存前缀，默认为api_helper，建议不同项目设置不同前缀以防止冲突

cache_ttl // 缓存时长，默认120分钟，可根据项目需求自行设置

api_base_url // 接口请求基础地址，默认为当前<host_name>，一般情况下不需要配置，若存在内外网不通的情况，可设置为相应内网地址

namespaces // 指定生成Api文档命名空间，数组，key为group，value为namespace，请确保namspace之间没有交集，否则小集合将被忽略
```
注意：请确保所配置的命名空间下存在次级命名空间，否则将被忽略，次级命名空间将作为模块名，更深层次命名空间与次级命名空间同级，例：

namespaces 配置示例
```php
namespaces => [
    'App'  => 'App\Http\Controllers', // 生效
    'Test' => 'App\Http\Controllers\Test', // 无效
    'Api'  => 'Api\Controllers' // 生效
];
```
该配置中，「App」包含「Test」，「App」与「Api」同级，所以「Test」被忽略

代码目录结构
```php
|--app
    |--Http
        |--Controllers
            |--TestController // 无效
            |--Demo
                |--DemoController // 生效
                |--Deep
                    |--DeepController // 生效，与DemoController 同级
|--Api
    |--Controllers
        |--Xxx
            |--XxxController // 生效
```
以上配置，TestController被忽略，DemoController 生效，DeepController与DemoController同级生效，XxxController生效

## 设置缓存驱动
修改 .env 文件，将缓存驱动设置为redis（推荐）
```php
CACHE_DRIVER=redis
```

若本机没有安装redis，可暂时设置为array
```php
CACHE_DRIVER=array
```

## 异常处理（推荐）
在/app/Exceptions/Handler.php文件的render方法中添加以下方法进行异常处理
```php
// 表单验证异常处理
if ($exception instanceof \Illuminate\Validation\ValidationException) {
    return response()->json(['code' => 11, 'msg' => $exception->errors(), 'data' => null]);
}
```

## 使用示例
```php
use BrooksYang\LaravelApiHelper\Traits\ResponseHelper;

/**
 * 这里是api标题
 *
 * @return \Illuminate\Http\Response
 */
public function index(Request $request)
{
    // 支持以下三种方式接收表单参数
    $paramA = $request->input('param_a'); // 参数一说明
    $paramB = $request->get('param_b'); // 参数二说明
    $paramC = Input::get('param_c'); // 参数三说明
    
    // 支持接收表单文件（v1.3.0新增）
    $file = $request->file('upload_file');
    
    // 以下是返回内容，ResponseHelper封装了三种返回方法，不强制使用，可自定义返回数据结构
    return $this->jsonResponse(['test' => 'blablabla'], '操作成功'); // 方式一，操作成功，返回数据及提示信息
    return $this->errorResponse(['code' => 'xxx', 'msg' => '操作失败']); // 方式二，操作失败，返回错误码及错误消息
    return $this->msgResponse('操作成功'); // 方式三，操作成功，仅返回成功提示消息
}
```

## 注意
若设置了redis驱动，添加了新的api之后，需执行清除缓存操作（若缓存驱动为array，则跳过该步骤）
```php
php artisan cache:clear
```

## 访问地址
```php
http://<HOST_NAME>/api/docs
```

## TODO LIST
- [ ] 支持资源路由
- [ ] 返回参数自动生成文档
- [ ] v2.0版本考虑引入 JWT，作为默认用户认证
