# Расширение Query Builder'а

Одним из ключевых моментов MVC модели является инкапсуляция бизнес логики. Интерфейс построения
запросов в Jelly достаточно естественный и гибкий, однако построение запросов в контроллере может
показаться излишним.

Jelly предоставляет возможность расширения собственного query builder'а модели, позволяя очень
элегантно решить этоу проблему. Например, для того, чтобы определить активных пользователей
в лице тех, кто логинился в течение последних 3х месяцев, можно написать следующий код:

	$active_users = Jelly::select('user')->where('last_login', '>', strtotime('-3 month'))->execute();

Но помещать этот код непосредственно в контроллер не целесообразно, ибо это бизнес логика. Более того,
если в будущем потребуется изменение определения активного пользователя, то придётся искать в коде
этот запрос.

Вместо этого, можно инкапсулировать эту логику, создав для неё класс `Model_Builder_User` который
расширяет `Jelly_Builder`. Если Jelly обнаружит класс с подобным наименованием, то он всегда будет
возвращать его экземпляр вместо стандартного `Jelly_Builder` для любых запросов, строящихся на
модели `User`.

Теперь можно добавить объявление необходимой логики при построении запроса. Можно даже изменить
стандартное поведение query builder'а для причудливых нужд определённых моделей.

Наш пример с активными пользователями решается следующим образом:

	class Model_Builder_User extends Jelly_Builder
	{
		public function active()
		{
			return $this->where('last_login', '>', strtotime('-3 month'));
		}
	}

	// Теперь в контроллере можно написать следующее
	$active_users = Jelly::select('user')->active()->execute();

Определённые таким образом методы можно использовать последовательно при необходимости.

Можно пойти дальше. Скажем, для некоторых моделей требуется одинаковая дополнительная логика. В 
этом случае, можно определить абстрактный класс builder'а где можно будет описать требующуюся логику, 
который будет расширять `Jelly_Builder`, который, в свою очередь, сможет быть расширен классом
`Model_Builder_ModelName`.

[!!] **Заметка**: Любая модель, без соответствующего builder класса будет просто использовать
стандартные методы построения запроса, описанные в классе `Jelly_Builder`.