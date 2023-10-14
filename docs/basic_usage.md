# Basic usage

The reason we choose queues is that we want to run jobs in the background.

Here we will present how you can create your own job class, which you will later use in your own queue.

---

- [Create a job class](#creating-a-job-class)
- [Implement a job](#implement-a-job)
- [Sending job to the queue](#sending-job-to-the-queue)
- [Consuming the queue](#consuming-the-queue)


### Create a job class

Here with help comes a generator that will allow us to quickly get started. In our example we will create the `Email` class:

    php spark queue:job Email

The above command will create a file in `App\Jobs` namespace. Once we have our class, we need to add it to the `$jobHandlers` array, in the `Config\Queue` configuration file.

```php
// app/Config/Queue.php

// ...

/**
 * Your jobs handlers.
 */
public array $jobHandlers = [
    'email' => Email::class,
];

// ...
```

### Implement a job

One of the most popular tasks delegated to a queue is sending email messages. Therefore, in this example, we will just implement that.

```php
<?php

namespace App\Jobs;

use Exception;
use Michalsn\CodeIgniterQueue\BaseJob;
use Michalsn\CodeIgniterQueue\Interfaces\JobInterface;

class Email extends BaseJob implements JobInterface
{
    /**
     * @throws Exception
     */
    public function process()
    {
        $email  = service('email', null, false);
        $result = $email
            ->setTo('test@email.test')
            ->setSubject('My test email')
            ->setMessage($this->data['message'])
            ->send(false);

        if (! $result) {
            throw new Exception($email->printDebugger('headers'));
        }

        return $result;
    }
}
```

The method that handles the job is the `process` method. It is the one that is called when our job is executed.

You may be wondering what the `$this->data['message']` variable is all about. We'll explain that in detail in the next section, but for now it's important for you to remember that all the variables we pass to the Job class are always held in the `$this->data` variable.

### Sending job to the queue

Sending a task to the queue is very simple and comes down to a call:

```php
service('queue')->push('QueueName', 'jobName', ['array' => 'parameters']);
```

In our particular case, for the Email class, it might look like this:

```php
service('queue')->push('emails', 'email', ['message' => 'Email message goes here']);
```

That's it, we just added our Email job to the queue.

### Consuming the queue

Since we sent our sample job to queue `emails`, then we need to run the worker with the appropriate queue:

    php spark queue:work emails

Now we are going to consume jobs from the queue `emails`. This command has many parameters, but you can learn more about that at [commands](commands.md) page.