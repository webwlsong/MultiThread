MultiThread
===========

php基于pcntl和posix扩展实现的多进程框架。

##基本要求
PHP 5.2 + 扩展pcntl, posix
```
扩展安装：http://www.php.net/manual/zh/book.posix.php、http://www.php.net/manual/zh/book.pcntl.php
```

##有以下几种优点：
~~~
1、子进程数量的控制
2、钩子形式的运行子进程，方便代码实现
3、3种不同的方式导入任务（数组输入、运行次数输入、动态加载）
4、子进程运行结果的收集（3种收集方式：文件、System V message queue、redis，推荐使用redis）
5、支持对子进程返回结果处理，并通过钩子方式来定义自己的处理过程
~~~

##注意事项：
~~~
1、建议不开启子进程返回结果，因为这样需要使用中间件来传递结果影响效率，能在子进程中处理尽量在子进程中处理
2、使用System V message queue为中间件的注意队列的总大小和单个消息的大小，
   如果返回结果大小超过单个消息的大小将会发生消息失败
3、非要使用消息中间件，建议使用redis
4、_fork($arg) 方法需要有输入参数，在子进程任务输入方式是次数时，$arg为null
5、_addTask() 方法返回的一定为数组，里面的元素对应 _fork($arg) 中的$arg
6、在开启子进程结果处理方法时，MultiThread 的 run() 方法是不会有返回值
7、注意php需要的扩展是否安装，基本扩展需要有pcntl,posix。同时也注意中间件的扩展是否安装，如果使用的话。
~~~
============
##### *Example*
~~~
include('MultiThread.php');
include('ChildThread.php');

class test extends ChildThread{
    
    public function __construct($type){
        parent::__construct(array('msg_type'=>$type));
    }
    
    public function _fork($arg){
        usleep(100000);
//         echo $arg . "\n";
        return $arg;
    }
    
    public function _addTask(){
        static $i = 0;
        if($i < 100){
            $i++;
        }else{
            return array();
        }
        return array($i);
    }
    
    public function _processReturn($rs){
        echo __CLASS__.'->'.__FUNCTION__ . ":" . $rs . "\n";
    }

}

$type = 0;
// $type = 1;
// $type = 2;
$start_time = microtime(true);
$test = new test($type);
$arg = array ('http://baidu.com','http://163.com1','http://163.com2','http://163.com3',
        'http://163.com4','http://163.com5','http://163.com6','http://163.com7','http://163.com8',
        'http://163.com9','http://163.com10','http://163.com11');
//使用$arg传入任务
$thread = new MultiThread($test, $arg, 10, true);
//使用$arg传入执行次数
// $thread = new MultiThread($test, 20, 10, true, true);
//使用_addTask()方法导入任务
// $thread = new MultiThread($test, null, 10, true);
$data = $thread->run();
var_dump($data);
echo $type . ' | Run time:' , (microtime(true) - $start_time) , "\n";
~~~

