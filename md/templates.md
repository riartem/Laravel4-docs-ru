# Шаблоны

- [Шаблоны контроллеров](#controller-layouts)
- [Шаблоны Blade](#blade-templating)
- [Другие директивы Blade](#other-blade-control-structures)

<a name="controller-layouts"></a>
## Шаблоны контроллеров

Один из способов использования шаблонов в Laravel - в виде шаблонов контроллеров. Если в классе контроллера определить свойство `layout`, то указанный шаблон будет создан автоматически и будет использоваться при генерации ответа клиенту.

**Определение шаблона контроллера:**

	class UserController extends BaseController {

		/**
		 * Шаблон, который должен использоваться при ответе.
		 */
		protected $layout = 'layouts.master';

		/**
		 * Отображает профиль пользователя.
		 */
		public function showProfile()
		{
			$this->layout->content = View::make('user.profile');
		}

	}

<a name="blade-templating"></a>
## Шаблоны Blade

Blade - простой, но мощный шаблонизатор, входящий в состав Laravel. В отличии от шаблонов контроллеров Blade основан на концепции наследования шаблонов и секциях. Все шаблоны Blade должны иметь расширение `.blade.php`.

**Создание шаблона Blade:**

	<!-- Расположен в app/views/layouts/master.blade.php -->

	<html>
		<body>
			@section('sidebar')
				Это - главная боковая панель.
			@show

			<div class="container">
				@yield('content')
			</div>
		</body>
	</html>

**Использование шаблона Blade:**

	@extends('layouts.master')

	@section('sidebar')
		@parent

		<p>Этот элемент будет добавлен к главной боковой панели.</p>
	@stop

	@section('content')
		<p>Это - содержимое страницы.</p>
	@stop

Заметьте, что шаблоны, которые расширяют другой Blade-шаблон с помощью `extend`, просто перекрывают секции последнего. Старое (перекрытое) содержимое может быть выведено директивой  `@parent`.

Иногда - например, когда вы не уверены, что секция была определена - вам может понадобиться указать значение по умолчанию для директивы `@yield`. Вы можете передать его вторым аргументом:

	@yield('section', 'Default Content');

<a name="other-blade-control-structures"></a>
## Другие директивы Blade

**Echoing Data**

	Привет, {{ $name }}.

	Текущее время эпохи UNIX: {{ time() }}.

Конечно, весь пользовательский ввод должен быть экранирован или очищен. Для экранирования используйте тройные скобки:

	Привет, {{{ $name }}}.

> **Внимание:** будьте очень осторожны и экранируйте переменные, которые содержат ввод от пользователя. Всегда используйте тройные скобки, чтобы преобразовать HTML-сущности в переменной в текст.

> Как показывает практика, вместо экранирования только пользовательских переменных безопаснее экранировать весь вывод, делая исключения только в редких случаях. Некоторые альтернативные шаблонизаторы, такие как HTMLki, делают это автоматически - прим. пер.

**If**

	@if (count($records) === 1)
		Здесь есть одна запись!
	@elseif (count($records) > 1)
		Здесь есть много записей!
	@else
		Здесь нет записей!
	@endif

	@unless (Auth::check())
		Вы не вошли в систему.
	@endunless

**Циклы**

	@for ($i = 0; $i < 10; $i++)
		Текущее значение: {{ $i }}
	@endfor

	@foreach ($users as $user)
		<p>Это пользователь{{ $user->id }}</p>
	@endforeach

	@while (true)
		<p>Это будет длиться вечно.</p>
	@endwhile

**Подшаблоны**

	@include('view.name')
	
Вы также можете передать массив переменных во включаемый шаблон:
	
	@include('view.name', array('some'=>'data'))
	
**Перезапись секций**

По умолчанию, содержимое новой секции добавляется в конец содержимого старой (перекрытой) секции. Для полной перезаписи можно использовать директиву  `overwrite`:
	
	@extends('list.item.container')

	@section('list.item.content')
		<p>Это - элемент типа {{ $item->type }}</p>
	@overwrite

**Языковые строки**

	@lang('language.line')

	@choice('language.line', 1);

**Комментарии**

	{{-- Этот комментарий не будет включён в сгенерированный HTML --}}