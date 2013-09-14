# Очереди

- [Настройка](#configuration)
- [Основы использования](#basic-usage)
- [Добавление замыканий](#queueing-closures)
- [Работа сервера приёма очереди](#running-the-queue-listener)
- [Push-очереди](#push-queues)

<a name="configuration"></a>
## Настройка

В Laravel, компонент Queue предоставляет единое API для различных сервисов очередей. Очереди позволяют вам отложить выполнение времязатратной задачи, такой как отправка e-mail, на более позднее время, таким образом на порядок ускоряя загрузку (генерацию) страницы.

Настройки очередей хранятся в файле `app/config/queue.php`. В нём вы найдёте настройки для драйверов-связей, которые поставляются вместе с фреймворком: [Beanstalkd](http://kr.github.com/beanstalkd), [IronMQ](http://iron.io), [Amazon SQS](http://aws.amazon.com/sqs), а также синхронный драйвер (для локального использования).

Упомянутые выше драйвера имеют следующие зависимости:

- Beanstalkd: `pda/pheanstalk`
- Amazon SQS: `aws/aws-sdk-php`
- IronMQ: `iron-io/iron_mq`

<a name="basic-usage"></a>
## Основы использования

**Добавление новой задачи в очередь:**

	Queue::push('SendEmail', array('message' => $message));

Первый аргумент метода `Queue::push` - имя класса, который должен использоваться для обработки задачи. Второй аргумент - массив параметров, которые будут переданы обработчику.

**Регистрация обработчика задачи:**

	class SendEmail {

		public function fire($job, $data)
		{
			//
		}

	}

Заметьте, что `fire` - единственный обязательный метод этого класса; он получает экземпляр объект `Job` и массив данных, переданных при добавлении задачи в очередь.

Если вы хотите использовать какой-то другой метод вместо `fire` - передайте его имя при добавлении задачи.

**Задача с произвольным методом-обработчиком:**

	Queue::push('SendEmail@send', array('message' => $message));

Как только вы закончили обработку задачи она должна быть удалена из очереди - это можно сделать методом `delete` объекта `Job`.

**Удаление выполненной задачи:**

	public function fire($job, $data)
	{
		// Обработка задачи...

		$job->delete();
	}

Если вы хотите поместить задачу обратно в очередь - используйте метод `release`:

**Помещение задачи обратно в очередь:**

	public function fire($job, $data)
	{
		// Обработка задачи...

		$job->release();
	}

Вы также можете указать число секунд, после которого задача будет помещена обратно:

	$job->release(5);

Если во время обработки задания возникнет исключение, задание будет помещено обратно в очередь. Вы можете получить число сделанных попыток запуска задания методом `attempts`:

**Получение числа попыток запуска задания:**

	if ($job->attempts() > 3)
	{
		//
	}


**Получение идентификатора задачи:**

	$job->getJobId();

<a name="queueing-closures"></a>
## Добавление замыканий

Вы можете помещать в очередь и функции-замыкания. Это очень удобно для простых, быстрых задач, выполняющихся в очереди.

**Добавление замыкания в очередь:**

	Queue::push(function($job) use ($id)
	{
		Account::delete($id);

		$job->delete();
	});

> **Внимание:** константы `__DIR__` и `__FILE__` не должны использоваться в замыканиях.

При использовании [push-очередей](#push-queues) Iron.io, будьте особенно внимательны при добавлении замыканий. Конечная точка выполнения, получающая ваше сообщение, должна проверить входящую последовательность-ключ, чтобы удостовериться, что запрос действительно исходит от Iron.io. Например, ваша конечная push-точка может иметь адрес вида `https://yourapp.com/queue/receive?token=SecretToken` где значение `token` можно проверять перед собственно обработкой задачи.

<a name="running-the-queue-listener"></a>
## Работа сервера приёма очереди

Laravel включает в себя задание Artisan, которое будет выполнять новые задачи по мере их поступления. Вы можете запустить его командой `queue:listen`:

**Запуск сервера приёма:**

	php artisan queue:listen

Вы также можете указать, какое именно соединение должно прослушиваться:

	php artisan queue:listen connection

Заметьте, что когда это задание запущено оно будет продолжать работать, пока вы не остановите его вручную. Вы можете использовать монитор процессов, такой как [Supervisor](http://supervisord.org/), чтобы удостовериться, что задание продолжает работать.

Кроме этого, вы можете указать число секунд, в течении которых будут выполняться задачи.

**Указание числа секунд для работы сервера:**

	php artisan queue:listen --timeout=60

Для обработки только первой задачи можно использовать команду `queue:work`.

**Обработка только первой задачи в очереди:**

	php artisan queue:work

<a name="push-queues"></a>
## Push-очереди

Push-очереди дают вам доступ ко всем мощным возможностям, предоставляемым подсистемой очередей Laravel 4 без запуска серверов или фоновых программ. На текущий момент push-очереди поддерживает только драйвер [Iron.io](http://iron.io). Перед тем, как начать, создайте аккаунт и впишите его данные в `app/config/queue.php`.

После этого вы можете использовать команду `queue:subscribe` Artisan для регистрации URL конечной точки, которая будет получать добавляемые в очередь задачи.

**Регистрация подписчика push-очереди:**

	php artisan queue:subscribe queue_name http://foo.com/queue/receive

Теперь, когда вы войдёте в ваш профиль Iron, то увидите новую push-очередь и её URL подписки. Вы можете подписать любое число URL на одну очередь. Дальше создайте маршрут для вашей конечной точки `queue/receive` и пусть он возвращает результат вызова метода `Queue::marshal`:

	Route::post('queue/receive', function()
	{
		return Queue::marshal();
	});

Этот метод позаботится о вызове нужного класса-обработчика задачи. Для помещения задач в push-очередь просто используйте всё тот же метод `Queue::push`, который работает и для обычных очередей.