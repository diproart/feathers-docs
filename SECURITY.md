# Безопасность Feathers

Отношение к безопасности в Feathers очень серьезно. Код Feathers открыт на 100% и кто-угодно может проверить его и убедится в том, что приложение на Feathers не взломано и не скомпромитированно. Но как разработчик приложения вы сами отвечаете за любые нарушения безопасности. Мы делаем всё возможное, чтобы Feathers оставался безопасным настолько, насколько это возможно.

## Как сообщить о проблеме в безопасности?

In order to give the community time to respond and upgrade we strongly urge you report all security issues to us. Send us a PM in [Slack](http://slack.feathersjs.com) or email us at [hello@feathersjs.com](mailto:hello@feathersjs.com) with details and we will respond ASAP. Security issues always take precedence over bug fixes and feature work so we'll work with you to come up with a resolution and plan and document the issue on Github in the appropriate repo.

Issuing releases is typically very quick. Once an issue is resolved it is usually released immediately with the appropriate semantic version.

## Вопросы безопасности

Вот некоторые вещи о которых вам следует позаботится при разработке приложения, чтобы сделать его более безопасным.

* Предварительно обрабатывать \(escape\) любой HTML и JavaScript для предотвращения XSS атак.
* Предварительно подготавливать любой SQL код \(обычно это делает SQL-библиотека\) для предотвращения SQL-инъекций.
* Events are sent by default to any client listening for that event. Lock down any private events that should not be broadcast by adding [filters](http://docs.feathersjs.com/real-time/filtering.html). Feathers authentication does this for all auth services by default.
* JSON Web Tokens \(JWT's\) are only signed, they are **not** encrypted. Therefore, the payload can be examined on the client. This is by design. **DO NOT** put anything that should be private in the JWT `payload` unless you encrypt it first.
* Не используйте слабый `secret`  для токенов. Генератор создает сложный токен автоматически. Нет необходимости  его менять.
* Используйте хуки для проверки ролей и прав доступа пользователей, чтобы быть уверенными, что доступ предоставлен правильно. Чтобы сделать процесс проверки проще мы создали [хуки авторизации](http://docs.feathersjs.com/authorization/bundled-hooks.html) (многие из которых добавляются на этапе генерации приложения или сервиса).

## Некоторые из технологий которые мы добавили

* Password storage inside `feathers-authentication` uses [bcrypt](https://github.com/dcodeIO/bcrypt.js). We don't store the salts separately since they are included in the bcrypt hashes.
* [JWT](https://jwt.io/) is used instead of cookies to avoid CSRF attacks. We use the `HS512` algorithm by default \(HMAC using SHA-512 hash algorithm\).
* We run [nsp](https://github.com/nodesecurity/nsp) as part of our CI. This notifies us if we are susceptible to any vulernabilites that have been reported to the [Node Security Project](https://nodesecurity.io/).

## XSS Атаки

As with any web application **you** need to guard against XSS attacks. Since Feathers persists the JWT in localstorage in the browser, if your app falls victim to a XSS attack your JWT could be used by an attacker to make malicious requests on your behalf. This is far from ideal. Therefore you need to take extra care in preventing XSS attacks. Our stance on this particular attack vector is that if you are susceptible to XSS attacks then a compromised JWT is the least of your worries because keystrokes could be logged and attackers can just steal passwords, credit card numbers, directly etc.

For more information see:

* [this issue](https://github.com/feathersjs/feathers-authentication/issues/132)
* and [this Auth0 forum thread](https://ask.auth0.com/t/stealing-jwt-from-authenticated-user/352/3).



