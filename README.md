Queue component for Yii2
====================
This component provides simple queue wrapper

Requirements
------------

[Redis](http://redis.io)
[yii2-redis](https://github.com/yiisoft/yii2-redis)

Installation
------------

The preferred way to install this extension is through [composer](http://getcomposer.org/download/).

Either run

```
php composer.phar require --prefer-dist atlasmobile/yii2-queue "*"
```

or add

```
"atlas/yii2-queue": "*"
```

to the require section of your `composer.json` file.


Usage
-----

To use this extension, simply add the following code in your application configuration:

```php
return [
    //....
    'components' => [
        'queue' => [
            'class' => \atlasmobile\queue\RedisQueue::class,
        ],
        'redis' => [
            'class' => \yii\redis\Connection::class,
            'hostname' => 'localhost',
            'port' => 6379,
            'database' => 0
        ],
    ],
];
```



First create a Job process class

```php
namespace console\jobs;

class MyJob implements \atlasmobile\queue\QueueHandler
{
    public function run(\atlasmobile\queue\Job $job, $data)
    {
        //process $data;
        var_dump($data);
    }
} 
```

OR

```php
namespace console\jobs;

class MyJob extends \atlasmobile\queue\BaseTask
{
	public function beforeRun(Job $job, QueuePayload $payload = []) {
		//todo before running task
	}

    public function run(\atlasmobile\queue\Job $job, $data)
    {
        //process $data;
        var_dump($data);
    }
    
    public function afterFun(Job $job, QueuePayload $payload = []) {
    	//todo after running task
    }
    
    public function onFail(Job $job, QueuePayload $payload = [], \Exception $exception) {
    	//todo what to do on fail running task
    }
} 
```




and than just push job to queue

```php

You can use component directly or static method to push job to queue: \atlasmobile\queue\helpers\Queue::push($job, $data = null, $queue = 'default', $options = [])

// Push job to the default queue and execute "run" method
Yii::$app->queue->push(\console\jobs\MyJob::class, ['a', 'b', 'c']); 

// or push it and execute any other method
Yii::$app->queue->push(\console\jobs\MyJob@myMethod::class, ['a', 'b', 'c']);

// or push it to some specific queue
Yii::$app->queue->push(\console\jobs\MyJob::class, ['a', 'b', 'c'], 'myQueue');

// or both
Yii::$app->queue->push(\console\jobs\MyJob@myMethod::class, ['a', 'b', 'c'], 'myQueue');

```


Map console controller in your app config

```php
return [
    ...
    'controllerMap' => [
        'queue' => \atlasmobile\queue\console\controllers\QueueController::class,
    ],
    ...
];
```

Examples of using queue controller:

```
# Process a first job from default queue and than exit the process
./yii queue/work

# continuously process jobs from default queue
./yii queue/listen

# process a job from specific queue and than exit the process
./yii queue/work --queue=queueName

# continuously process jobs from specific queue
./yii queue/listen --queue=myQueue

```

Also you can store failed jobs into db

```bash
./yii queue/table-failed //will migrate table for failed jobs

./yii queue/listen --storeFailedJobs=true

```

Then after some time when table will filled with failed jobs, do next:

```bash 
./yii queue/failed 
```

This command will add to queue all failed jobs in FIFO order
