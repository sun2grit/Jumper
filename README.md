# Jumper
Simple &amp;Strong Message Message Queue Framework

【Jumper消息队列服务框架】

Jumper消息队列服务分成服务端和客户端

________________________________________________________________________

【服务端】
服务端是分别使用Python和PHP实现
Coop为队列容器，当前默认为10进程，抢占式执行
Jumper为队列框架，暂时支持RabbitMQ消息队列服务。基础框架以ThinkPHP为原型进行重新设计，能很好的兼容ThinkPHP5.0的框架环境。
其中消息接收控制器放在application应用目录，与controller同一级的worker目录下,接收控制器需继承于Worker
每个消息方法都有固定的Task $task参数，在消息方法中可以引用应用中的模型，服务等类库及函数。

【示例程序】

/**
 * @package jumper\QueueServer
 *
 * 消息接收控制器
 *
 * @author: Asun
 * @version 1.0
 * @copyright (c) 2006~2017 Sltech All rights reserved.
 */
namespace app\artisan\worker;

use app\common\service\UserService;
use jumper\Worker;
use jumper\Task;

/**
 * 消息接收控制器
 * @package app\artisan\worker
 */
class Order extends Worker
{
	public function test(Task $task)
    {
        echo "Order.test{$task->index}\n";
    }
	public function at(Task $task)
    {
        echo "Order.at{$task->index}\n";
    }
    public function delay(Task $task)
    {
        echo $task->toJson().PHP_EOL;
		echo "Order.delay {$task->times}\n";
		//返回True才会继续循环执行
        return true;
    }
}


________________________________________________________________________

【客户端】
客户端在使用时需要将以下文件配置成自动加载
extend/Queue.php

在PHP程序在发布消息需要在头部引用 Queue
use extend/Queue
然后在其它需要调用的地方可以直接使用静态函数生成队列消息并发送

发送实时消息
Queue::sendNow('queue.order.test',array('data'=>'ttt'));

发送定时消息
Queue::sendAt('queue.order.at',Now()+10000,array('order_id'=>'123'));

发送循环消息
Queue::sendDelay('queue.order.delay',10000,1000,array('order_id'=>'123'));

【示例程序】

namespace app\artisan\controller;

use extend\Queue;
use think\Controller;

use sltech\Session;

/**
 * 队列消息发送示例
 * @package app\artisan\controller
 */
class QueueTest extends Controller
{
	public function index()
	{
		for($i=0;$i<1;$i++){
			Queue::sendNow('artisan.order.test',array('data'=>'ttt'));
		}

		return view('welcome');
	}
}
