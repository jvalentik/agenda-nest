# Agenda Nest

<a href="https://www.npmjs.com/~agenda-nest" target="_blank"><img src="https://img.shields.io/npm/v/agenda-nest.svg" alt="NPM Version" /></a>
<a href="https://www.npmjs.com/~agenda-nest" target="_blank"><img src="https://img.shields.io/npm/l/agenda-nest.svg" alt="Package License" /></a>
</p>

A NestJS module for Agenda

> ⚠️ This package is not yet stable and is subject breaking changes until such time as v1.0.0 is released.

## Table of Contents
- [Background](#background)
- [Install](#install)
- [Configure Agenda](#configure-agenda)
- [Job processors](#job-processors)
- [Job schedulers](#job-schedulers)
- [Event listeners](#event-listeners)
- [Manually working with the queue](#manually-working-with-the-queue)
- [License](#license)

## Background

Agenda Nest provides a NestJS module wrapper for [Agenda](https://github.com/agenda/agenda), a light-weight job scheduling library.  Heavily inspired by Nest's own Bull implementation, [@nestjs/bull](https://github.com/nestjs/bull), Agenda Nest provides a fully-featured implementation, complete with decorators for defining your jobs, processors and queue event listeners.  You may optionally, make use of Agenda Nest's Express controller to interface with your queues through HTTP.

### Dependencies

Agenda uses MongoDB to persist job data, so you'll need to have Mongo (or mongoose) installed on your system.

## Install

```bash
npm install agenda-nest
```

## Configure Agenda

As Agenda Nest is a wrapper for Agenda, it is configurable with same properties as the Agenda instance. Refer to [AgendaConfig](https://github.com/agenda/agenda/blob/master/lib/agenda/index.ts#L39) for the complete configuration type.

```js
import { AgendaModule } from 'agenda-nest';

@Module({
  imports: [
    AgendaModule.register({
      db: {
        addresss: 'mongodb://localhost:27017',
        collection: 'job-queue',
      },
    }),
  ],
  providers: [Jobs],
})
export class AppModule {}
```

## Job processors

Job processors are defined using the `@Define` decorator.  Refer to Agenda's documentation on job definition options.

```js
@Injectable()
export class Jobs {
  @Define('say hello')
  sayHello(job: Job) {
    this.logger.log(`Hello  ${job.attrs.userName}!`)
  }

  @Define('some long running job')
  async handleSomeLengthyTask(job: Job) {
    const data = await doSomeLengthyTask();

    await formatThatData(data);
    await sendThatData(data);
  }
}

```

## Job schedulers

### `@Every(nameOrOptions: string | JobOptions)`

Defines a job to run at the given interval

```js
@Injectable()
export class Jobs {
  @Every('15 minutes')
  async printAnalyticsReport(job: Job) {
    const users = await User.doSomethingReallyIntensive();
    processUserData(users);
  }

  @Every({ name: 'send notifications', interval: '15 minutes' })
  async sendNotifications(job: Job) {
    const users = await User.doSomethingReallyIntensive();
    sendNotification(users, "Welcome!");
  }
}

```

### `@Schedule(nameOrOptions: string | JobOptions)`

Schedules a job to run once at the given time.

```js
@Injectable()
export class Jobs {
  @Schedule('tomorrow at noon')
  async printAnalyticsReport(job: Job) {
    const users = await User.doSomethingReallyIntensive();
    processUserData(users);
  }

  @Scheduler({ name: 'send notifications', when: 'tomorrow at noon' })
  async sendNotifications(job: Job) {
    const users = await User.doSomethingReallyIntensive();
    sendNotification(users, "Welcome!");
  }
}

```

### `@Now(name?: string)`

Schedules a job to run once immediately.

```js
@Injectable()
export class Jobs {
  @Now()
  async doTheHokeyPokey(job: Job) {
    hokeyPokey();
  }

  @Now('do the cha-cha')
  async doTheChaCha(job: Job) {
    chaCha();
  }
}

```

## Event Listeners

Agenda generates a set of useful events when queue and/or job state changes occur. Agenda NestJS provides a set of decorators that allow subscribing to a core set of standard events.

Event listeners must be declared within an injectable class (i.e., within a class decorated with the @Queue() decorator). To listen for an event, use one of the decorators in the table below to declare a handler for the event. For example, to listen to the event emitted when a job enters the active state in the audio queue, use the following construct:

```js
import { OnQueueReady } from 'agenda-nest';
import { Job } from 'agenda';

@Queue()
export class JobsQueue {
  @OnQueueReady()
  onReady() {
    console.log('Jobs queue is ready to run our jobs');
  }
  ...
```

### Agenda Events

An instance of an agenda will emit the queue events listed below. Use the corresponding method decorator to listen for and handle each event.

| Event   | Listener          |                                                                                |
|:--------|:------------------|:-------------------------------------------------------------------------------|
| `ready` | `@OnQueueReady()` | called when Agenda mongo connection is successfully opened and indices created |
| `error` | `@OnQueueError()` | called when Agenda mongo connection process has thrown an error                |

### Job Queue Events

An instance of an agenda will emit the job events listed below. Use the corresponding method decorator to listen for and handle each event.

| Event                             | Listener                        |                                                                   |
|:----------------------------------|:--------------------------------|:------------------------------------------------------------------|
| `start` or `start:job name`       | `@OnJobStart(name?: string)`    | called just before a job starts                                   |
| `complete` or `complete:job name` | `@OnJobComplete(name?: string)` | called when a job finishes, regardless of if it succeeds or fails |
| `success` or `success:job name`   | `@OnJobSuccess(name?: string)`  | called when a job finishes successfully                           |
| `fail` or `fail:job name`         | `@OnJobFail(name?: string)`     | called when a job throws an error                                 |

## License

Agenda Nest is [MIT licensed](https://github.com/jongolden/agenda-nest/blob/main/LICENSE).
