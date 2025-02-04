---
id: version-4.4.1-authentification
title: Authentification
original_id: authentification
---

Аутентификација је везана за auth [plugin](plugins.md) који користите. Рестрикције које се односе на пакет се могу подесити (handle) преко [Package Access](packages.md).

<div id="codefund">''</div>

Аутентификација клијента се обавља (handle) преко самог `npm` клијента. Након што се пријавите у апликацију:

```bash
npm adduser --registry http://localhost:4873
```

Токен се генерише у фајлу за конфигурацију `npm` који се налази у home фолдеру корисника. Како бисте сазнали више о `.npmrc` прочитајте [official documentation](https://docs.npmjs.com/files/npmrc).

```bash
cat .npmrc
registry=http://localhost:5555/
//localhost:5555/:_authToken="secretVerdaccioToken"
//registry.npmjs.org/:_authToken=secretNpmjsToken
```

#### Анонимно публиковање

`verdaccio` Вам омогућава да пружите могућност анонимног публиковања. Како бисте успели у томе, потребно је да подесите [packages access](packages.md).

Пример:

```yaml
  'my-company-*':
    access: $anonymous
    publish: $anonymous
    proxy: npmjs
```

As is described [on issue #212](https://github.com/verdaccio/verdaccio/issues/212#issuecomment-308578500) until `npm@5.3.0` and all minor releases **won't allow you publish without a token**.

## Understanding Groups

### The meaning of `$all` and `$anonymous`

As you know *Verdaccio* uses the `htpasswd` by default. That plugin does not implement the methods `allow_access`, `allow_publish` and `allow_unpublish`. Thus, *Verdaccio* will handle that in the following way:

* If you are not logged in (you are anonymous), `$all` and `$anonymous` means exactly the same.
* If you are logged in, `$anonymous` won't be part of your groups and `$all` will match any logged user. A new group `$authenticated` will be added to the list.

As a takeaway, `$all` **will match all users, independently whether is logged or not**.

**The previous behavior only applies to the default authentication plugin**. If you are using a custom plugin and such plugin implements `allow_access`, `allow_publish` or `allow_unpublish`, the resolution of the access depends on the plugin itself. Verdaccio will only set the default groups.

Let's recap:

* **logged**: `$all`, `$authenticated`, + groups added by the plugin
* **anonymous (logged out)**: `$all` and `$anonymous`.

## Подразумевана htpasswd

Како би се поједноставио setup, `verdaccio` користи plugin базиран на `htpasswd`. Since version v3.0.x the `verdaccio-htpasswd` plugin is used by default.

```yaml
auth:
  htpasswd:
    file: ./htpasswd
    # Максимални број корисника који могу да се региструју, фабрички је подешено на бесконачно "+inf".
    # Ако хоћете да онемогућите регистрацију, подесите ово на -1.
    #max_users: 1000
```

| Својство  | Тип    | Неопходно | Пример     | Подршка | Опис                                   |
| --------- | ------ | --------- | ---------- | ------- | -------------------------------------- |
| file      | string | Да        | ./htpasswd | all     | фајл који садржи шифроване credentials |
| max_users | number | Не        | 1000       | all     | подешава максимални број корисника     |

Ако се одлучите на то да не дозволите корисницима да се пријаве, можете подесити `max_users: -1`.
