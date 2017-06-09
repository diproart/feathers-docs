# Хуки аутентификации

[![GitHub stars](https://img.shields.io/github/stars/feathersjs/feathers-authentication-hooks.png?style=social&label=Star)](https://github.com/feathersjs/feathers-authentication-hooks/)
[![npm version](https://img.shields.io/npm/v/feathers-authentication-hooks.png?style=flat-square)](https://www.npmjs.com/package/feathers-authentication-hooks)
[![Changelog](https://img.shields.io/badge/changelog-.md-blue.png?style=flat-square)](https://github.com/feathersjs/feathers-authentication-hooks/blob/master/CHANGELOG.md)

```
$ npm install feathers-authentication-hooks --save
```

`feathers-authentication-hooks` - это пакет, который содержит некоторые полезные хуки для аутентификации и авторизации. Для дополнительной информации  см [главу о хуках](../hooks.md). 

> **Замечание:** Хуки аутентификации выполняются только в том случае, когда в установлено значение `params.provider` (когда доступ к методу осуществляется извне, например через [REST](../rest.md) or [Socketio](../socketio.md)).


## queryWithCurrentUser

`queryWithCurrentUser` **before** хук автоматически добавляет к запросу идентификатор пользователя как параметр `id`. Это удобно когда вы хотите возвратить данные, например "сообщения", которые были добавлены конкретно текущим пользователем.

```js
const hooks = require('feathers-authentication-hooks');

app.service('messages').before({
  find: [
    hooks.queryWithCurrentUser({ idField: 'id', as: 'sentBy' })
  ]
});
```

#### Options

- `idField` (default: '_id') [optional] - Название поля `id` для объекта пользователя.
- `as` (default: 'userId') [optional] - Название поля в запрашиваемом ресурсе, соответствующее полю `id` пользователя.

Когда мы используем это хук с опциями по умолчанию, то `User._id` будет скопировано  в `hook.params.query.userId`.

## restrictToOwner

`restrictToOwner` предназначен для использования как **before** хук. Он позволяет пользователю получать и изменять ресурсы принадлежащие только ему самому. Если доступа нет будет возвращена ошибка доступа _Forbidden_. Хук может использоваться с любым методом.

Для метода `find` и для `patch`, `update`, `remove` для многих записей (с `id` установленным  в `null`), будет вызван хук [queryWithCurrentUser](#queryWithCurrentUser) для ограничений к текущему пользователю. Для всех остальных случаев, перед продолжением будет извлекаться запись и проверятся, что она принадлежит пользователю. 

```js
const hooks = require('feathers-authentication-hooks');

app.service('messages').before({
  remove: [
    hooks.restrictToOwner({ idField: 'id', ownerField: 'sentBy' })
  ]
});
```

#### Options

- `idField` (default: '_id') [optional] - Название поля `id` для объекта пользователя.
- `ownerField` (default: 'userId') [optional] - Название поля в запрашиваемом ресурсе, соответствующее полю `id` пользователя.


## restrictToAuthenticated

Хук `restrictToAuthenticated` вызывает ошибку если доступ пользователь не авторизован, проверяя объект `hook.params.user`. Хук может использоваться в любом методе сервиса. Он предназначен для использования в хуке **before** и не имеет никаких аргументов.

```js
const hooks = require('feathers-authentication-hooks');

app.service('user').before({
  get: [
    hooks.restrictToAuthenticated()
  ]
});
```

#### Options

- `entity` (default: 'user') [optional] - Название свойства в `hook.params` для проверки


## associateCurrentUser

**before** хук `associateCurrentUser` похож на `queryWithCurrentUser`, но работает для входящего параметра **data**, а не  параметров **query**. Он полезен для того, чтобы автоматически добавлять `id` текущего пользователя к создаваемым ресурсам. Может использоваться для `create`, `update`, or `patch` методов.

```js
const hooks = require('feathers-authentication-hooks');

app.service('messages').before({
  create: [
    hooks.associateCurrentUser({ idField: 'id', as: 'sentBy' })
  ]
});
```

#### Options

- `idField` (default: '_id') [optional] - Название поля `id` для объекта пользователя.
- `as` (default: 'userId') [optional] - Поле, которое будет заполнено идентификатором пользователя в создаваемом ресурсе.


## restrictToRoles

Хук `restrictToRoles` предназначен для использования как **before** хук. Он разрешает пользователю получать ресурсы которыми он владеет или ресурсы принадлежащие к определенным ролям. Хук возвратит ошибку _Forbidden_ если у пользователя нет необходимых прав.  Может использоваться в методах `all` когда опция **owner**  установлена  в 'false'.  Когда опция **owner** имеет значение `true` хук может использоваться только в `get`, `update`, `patch`, и `remove` методах сервиса.

```js
const hooks = require('feathers-authentication-hooks');

app.service('messages').before({
  remove: [
    hooks.restrictToRoles({
        roles: ['admin', 'super-admin'],
        fieldName: 'permissions',
        idField: 'id',
        ownerField: 'sentBy',
        owner: true
    })
  ]
});
```

#### Options

- `roles` (**required**) - An array of roles that a user must have at least one of in order to access the resource.
- `fieldName` (default: 'roles') [optional] - The field on your user object that denotes their roles.
- `idField` (default: '_id') [optional] - The id field on your user object.
- `ownerField` (default: 'userId') [optional] - The id field for a user on your resource.
- `owner` (default: 'false') [optional] - Denotes whether it should also allow owners regardless of their role (ie. the user has the role **or** is an owner).


## hasRoleOrRestrict

`hasRoleOrRestrict` is meant to be used as a **before** hook for any service on the **find** or **get** methods. Unless the user has one of the roles provided, it will add a restriction onto the query to limit what resources return.

```js
const hooks = require('feathers-authentication-hooks');

app.service('messages').before({
  find: [
    hooks.hasRoleOrRestrict({
        roles: ['admin', 'super-admin'],
        fieldName: 'permissions',
        restrict: { approved: true }
    })
  ]
});
```

#### Options

- `roles` (**required**) - An array of roles that a user must have at least one of in order to access the resource.
- `fieldName` (default: 'roles') [optional] - The field on your user object that denotes their roles.
- `restrict` (default: undefined) - The query to merge into the client query to limit what resources are accessed
