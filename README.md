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
composer require --prefer-dist "atlasmobile/yii2-queue=*"
```

or add

```
"atlas/yii2-queue": "*"
```

to the require section of your `composer.json` file.



Application configuration
-------------------------

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
        
        'controllerMap' => [
            'queue' => \atlasmobile\queue\console\controllers\QueueController::class,
        ],
    ],
];
```


First Job
---------

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
	public function beforeRun(Job $job, QueuePayload $payload) {
		//todo before running task
	}

    public function run(\atlasmobile\queue\Job $job, $data)
    {
        //process $data;
        var_dump($data);
    }
    
    public function afterRun(Job $job, QueuePayload $payload) {
    	//todo after running task
    }
    
    public function onFail(Job $job, QueuePayload $payload, \Exception $exception) {
    	//todo what to do on fail running task
    }
} 
```




and than just push job to queue

```php

// You can use component directly or static method to push job to queue: 
\atlasmobile\queue\helpers\Queue::push($job, $data = null, $queue = 'default', $options = [])

// Push job to the default queue and execute "run" method
Yii::$app->queue->push(\console\jobs\MyJob::class, ['a', 'b', 'c']); 

// or push it and execute any other method
Yii::$app->queue->push('\console\jobs\MyJob@myMethod', ['a', 'b', 'c']);

// or push it to some specific queue
Yii::$app->queue->push(\console\jobs\MyJob::class, ['a', 'b', 'c'], 'myQueue');

// or both
Yii::$app->queue->push('\console\jobs\MyJob@myMethod', ['a', 'b', 'c'], 'myQueue');

```

#### Since 1.0.5 available delayed tasks
```php

// Just a string
Yii::$app->queue->pushDelayed(\console\jobs\MyJob::class, '+3 hours', ['a', 'b', 'c']);
 
// Or \DateTime
$dt = new \DateTime('now');
$dt->modify('+1 week')->modify('+3 days');
Yii::$app->queue->pushDelayed(\console\jobs\MyJob::class, $dt, ['a', 'b', 'c']);


// Or oldschool
Yii::$app->queue->pushDelayed(\console\jobs\MyJob::class, time() + 86400, ['a', 'b', 'c']);

```



Listener
--------

### If you wanna use supervisor, put this config:

```ini
[program:yiiqueue]
command=php /path/to/project/yii queue/listen
process_name=%(program_name)s_%(process_num)02d
numprocs=4  ; customize workers
directory=/path/to/project
autostart=true
autorestart=true
user=nginx ; executor user
stdout_logfile=/path/to/project/runtime/logs/queue.out.log
stdout_logfile_maxbytes=10MB
stderr_logfile=/path/to/project/runtime/logs/queue.err.log
stderr_logfile_maxbytes=10MB
```

### Queue listener examples:
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


### Delayed listener examples (since 1.0.5):
```bash
# Pop list of delayed jobs and if "time to work", puts them to queue
./yii queue/delayed

# Or for supervisor or just for "&"
./yii queue/listen-delayed
```

Method listen-delayed by defaults check new delayed jobs every 30 seconds, but u can set --pollFreqSecs=MY_SECONDS


### Also you can store failed jobs into db

First, run migration to create table with failed jobs
```
 ./yii queue/failed-table
```

and run 
``` 
./yii queue/listen --storeFailedJobs=true
```

Then after some time when table will filled with failed jobs, do next:

```bash 
./yii queue/failed 
```
This command will add to queue all failed jobs in [FIFO](https://en.wikipedia.org/wiki/FIFO_and_LIFO_accounting) (first-in-first-out) order


To flush table with failed jobs:
```bash
./yii queue/failed-flush
```

### Monitor
In some cases you may want to see how much and/or what jobs are queued now.
To see basic information, use:

```bash
./yii queue/monitor
```

