# NBA Task Queue Deployment

> Code and Structure to deploy scripts and tasks within a task queue paradigm and data pipeline.

I need a application structure setup which uses celery + RabbitMQ for task distribution and workers management. I have a web application that uses machine learning and deep learning algorithms to do predictions from NBA. These predictive models are deployed on this server as python modules and their predictive functions are integrated with celery as individual task workers.

For example, X (user) wants to know predict the range of a players score for an upcoming game and submits query to the web application. The web application will initiate celery tasks with X's query payload. its a pipeline of requests and task where Celery tasks after performing certain operations submits to another Celery task which runs the machine learning / deep learning's ML function, and then the web application starts waiting for the Celery server tasks to be completed and response to be received.

## Features

- [Celery](https://docs.celeryproject.org/en/stable/index.html) is a simple, flexible, and reliable distributed system to process vast amounts of messages, while providing operations with the tools required to maintain such a system.
- [RabbitMQ](https://www.rabbitmq.com/) is the most widely deployed open source message broker.

### Workings

Data-heavy tasks, like the NBA machine learning models we use or the automated scraping and cleaning of fresh data, may take many seconds or even minutes to complete. This leads to X (user) having to wait for the result and we assume most users want a pretty streamlined experience.

i want to add a layer where our api server takes requests gives automatic feedback to the user that we are currently processing their request. There are a couple of ways to creating these systems to create these instant feedback experiences; I've chosen to create the task server using is Celery and it’s platform.

With Celery you can run different tasks simultaneously using the main process, and while you do your job, Celery will complete the smaller tasks at hand. You can set up queues, work with data chunks on long-running tasks, and set up times for your tasks to be executed. Tasks can do whatever we program them to do, update the db, run a corn job to check the servers status, and what we are aiming for ... running our ML model functions.

You use Celery to accomplish a few main goals:

• Define independent tasks that your workers can do as a Python function.
• These tasks are published to a broker we'll be using RabbitMQ
• Celery listens to a message broker (RabbitMQ) to get new task requests.
• Then assigns those requests to workers to complete the task which celery handles.
• We monitor the progress and status of tasks and workers by having the Web Application (Flask) subscribe to the celery updates.
• The the finalized task results are inserted into our PostgreSQL result backend.

#### Defined Terms

Broker
The Broker (RabbitMQ) is responsible for the creation of task queues, dispatching tasks to task queues according to some routing rules, and then delivering tasks from task queues to workers.

Consumer
The Consumer (Celery Workers) is the one or multiple Celery workers executing the tasks. You could start many workers depending on your use case.

Result Backend
The Result Backend is used for storing the results of your tasks. However, it is not a required element, so if you do not include it in your settings, you cannot access the results of your tasks.

## Installation

### Running the unit tests on scripts

### Starting a Worker Process

### Starting the Web Scraping Process

### Starting the ETL Process

## Example

## Todo Checklist

A helpful checklist to gauge how your README is coming on what I would like to finish:

- [ ] ?

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

[MIT](https://choosealicense.com/licenses/mit/)
