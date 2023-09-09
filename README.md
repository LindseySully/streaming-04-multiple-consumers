# streaming-04-multiple-consumers

> Use RabbitMQ to distribute tasks to multiple workers

One process will create task messages. Multiple worker processes will share the work. 


## Before You Begin

1. Fork this starter repo into your GitHub.
1. Clone your repo down to your machine.
1. View / Command Palette - then Python: Select Interpreter
1. Select your conda environment. 

## Read

1. Read the [RabbitMQ Tutorial - Work Queues](https://www.rvabbitmq.com/tutorials/tutorial-two-python.html)
1. Read the code and comments in this repo.

## RabbitMQ Admin 

RabbitMQ comes with an admin panel. When you run the task emitter, reply y to open it. 

(Python makes it easy to open a web page - see the code to learn how.)

## Execute the Producer

1. Run emitter_of_tasks.py (say y to monitor RabbitMQ queues)
    - In order to get the website to open correctly:
        - `rabbitmq-plugins enable rabbitmq_management`
Explore the RabbitMQ website.


## Execute a Consumer / Worker

1. Run listening_worker.py

Will it terminate on its own? How do you know? 
- The program will not terminate on it's own. In the code it specifically waits for the user to input CTRL + C.
- This is identified in the program via the KeyboardInterrupt which triggers the sys.exit() function.

## Ready for Work

1. Use your emitter_of_tasks to produce more task messages.

## Start Another Listening Worker 

1. Use your listening_worker.py script to launch a second worker. 

Follow the tutorial. 
Add multiple tasks (e.g. First message, Second message, etc.)
How are tasks distributed? 
Monitor the windows with at least two workers. 
Which worker gets which tasks?
- If one worker is busy with a message, than the second worker will receive the new message, however; it does alternate the messages between both workers as well. 
    - The screenshot shows that Worker #1 (top right terminal) received message 1 & 3.
    - Worker #2 (bottom right terminal) received message 2 & 5

## Notes
#### Durable Queues & Messages
- Durable queues/messages mean that if RabbitMQ quits or crashes it will make sure that the messages aren't lost.
    - Both the message and queue need to be marked as durable for this to work appropriately.
    1. Queue:
        - To ensure the queue will survive a node restart:
        `channel.queue_declare(queue = "queue_name",durable =True)`
        - This needs to be added to **both** the producer and consumer nodes
    1. Message:
        - To ensure the messages will survive the messages need to be marked as **persistant**
        `delivery_mode = pika.spec.PERSISTANT_DELIVERY_MODE`
        - Note on message persistence
            - Marking messages as persistent doesn't fully guarantee that a message won't be lost. Although it tells RabbitMQ to save the message to disk, there is still a short time window when RabbitMQ has accepted a message and hasn't saved it yet. 
            - Also, RabbitMQ doesn't do fsync(2) for every message -- it may be just saved to cache and not really written to the disk. The persistence guarantees aren't strong, but it's more than enough for our simple task queue. If you need a stronger guarantee then you can use publisher confirms.

#### Message Acknowledgements
- In order to make sure a message is never lost, RabbitMQ supports message acknowledgments. An ack(nowledgement) is sent back by the consumer to tell RabbitMQ that a particular message had been received, processed and that RabbitMQ is free to delete it.
    - A timeout (30 minutes by default) is enforced on consumer delivery acknowledgement. This helps detect buggy (stuck) consumers that never acknowledge deliveries. You can increase this timeout as described in Delivery Acknowledgement Timeout.
    - Manual message acknowledgments are turned on by default. In previous examples we explicitly turned them off via the `auto_ack=True` flag. 
        - Using this code, you can ensure that even if you terminate a worker using CTRL+C while it was processing a message, nothing is lost. Soon after the worker terminates, all unacknowledged messages are redelivered.
- Forgotten acknowledgment
    - It's a common mistake to miss the basic_ack. It's an easy error, but the consequences are serious. Messages will be redelivered when your client quits (which may look like random redelivery), but RabbitMQ will eat more and more memory as it won't be able to release any unacked messages.
    - In order to debug this kind of mistake you can use rabbitmqctl to print the messages_unacknowledged field:
        - `sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged`

## Reference

- [RabbitMQ Tutorial - Work Queues](https://www.rabbitmq.com/tutorials/tutorial-two-python.html)


## Screenshot

See a running example with at least 3 concurrent process windows here:
