# Домашнее задание к занятию "`Введение в Terraform`" - `Живарев Игорь`


### Задание 1

Задание 1
1. Перейдите в каталог src. Скачайте все необходимые зависимости, использованные в проекте.
2. Изучите файл .gitignore. В каком terraform файле согласно этому .gitignore допустимо сохранить личную, секретную информацию?
3. Выполните код проекта. Найдите в State-файле секретное содержимое созданного ресурса random_password, пришлите в качестве ответа конкретный ключ и его значение.
4. Раскомментируйте блок кода, примерно расположенный на строчках 29-42 файла main.tf. Выполните команду `terraform validate`. Объясните в чем заключаются намеренно допущенные ошибки? Исправьте их.
5. Выполните код. В качестве ответа приложите вывод команды `docker ps`
6. Замените имя docker-контейнера в блоке кода на `hello_world`, выполните команду `terraform apply -auto-approve`. Объясните своими словами, в чем может быть опасность применения ключа -auto-approve ? В качестве ответа дополнительно приложите вывод команды `docker ps`
7. Уничтожьте созданные ресурсы с помощью terraform. Убедитесь, что все ресурсы удалены. Приложите содержимое файла terraform.tfstate.
8. Объясните, почему при этом не был удален docker образ nginx:latest ? Ответ подкрепите выдержкой из документации провайдера.


Ответ: 

2. Личную, секретную информацию можно сохранить в файле `personal.auto.tfvars`;
3. Секретное содержимое созданного ресурса random_password `"result": "cVC8UNdUxio9uucz",`
![random_password](img/ter-01_01.png)
4. Первая ошибка говорит о том, что у чоздаваемого ресурса не указано угикальное имя, во-второй ошибке говорится, что уникальное имя содержит не допустимое значение (символ), третья ошибка - заменить 31 строку `name  = "example_${random_password.random_string_FAKE.resulT}"` на `name  = "example_${random_password.random_string.result}"`
```
╭─shaman@banner ~/Netology/github/ter-01-hw/src ‹main●› 
╰─$ terraform validate
╷
│ Error: Missing name for resource
│ 
│   on main.tf line 24, in resource "docker_image":
│   24: resource "docker_image" {
│ 
│ All resource blocks must have 2 labels (type, name).
╵
╷
│ Error: Invalid resource name
│ 
│   on main.tf line 29, in resource "docker_container" "1nginx":
│   29: resource "docker_container" "1nginx" {
│ 
│ A name must start with a letter or underscore and may contain only letters, digits, underscores, and dashes.

╷
│ Error: Unsupported attribute
│ 
│   on main.tf line 31, in resource "docker_container" "nginx":
│   31:   name  = "example_${random_password.random_string_FAKE.resulT}"
│ 
│ This object has no argument, nested block, or exported attribute named "resulT". Did you mean "result"?
```

![terraform_validate](img/ter-01_02.png)

5. Вывод команды `docker ps`

![docker ps](img/ter-01_03.png)

6. Опасность применения ключа `-auto-approve`, по моему мнению, заключается в отсутствии проверки за внесением измнений TERRAFORM в создоваемую, или, что ещё хуже, в изменяемую инфроструктуру. Если в файле конфигурации есть ошибки и/или неточности они без проверки и исправлений будут выполнены и применены, что повлечёт за собой последствия. Вывод команды `docker ps`

![docker ps](img/ter-01_04.png)

7. Выполнение комманды `terraform destroy`

![terraform destroy](img/ter-01_05.png)

Содержание файла `terraform.tfstate`
```
{
  "version": 4,
  "terraform_version": "1.4.5",
  "serial": 11,
  "lineage": "04a5199b-5f58-b6c0-27eb-6b7a6a0fb9d1",
  "outputs": {},
  "resources": [],
  "check_results": null
}
```

8. Образ docker nginx:latest не был удален из-за того что TERRAFORM удаляет только то что создал сам, в обратном порядке по отношению к комманде `terraform apply`. В документации к провайдеру говорится, что terraform не может удалить ресурсы запущеные другими сервисами.

```
Destroy
The terraform destroy command terminates resources managed by your Terraform project. This command is the inverse of terraform apply in that it terminates all the resources specified in your Terraform state. It does not destroy resources running elsewhere that are not managed by the current Terraform project.

Destroy the resources you created.
```

### Доработка ДЗ

В ответе на вопрос 1.7 неправильный скриншот, надеюсь подтянется верный.

Дополняю ответ на вопрос 1.8 после возврата ДЗ на доработку.
Возможно terraform не стал удалять docker image из-за ключа `keep_locally = true`. Интернеты по данному вопросу говорят следующее:

```
keep_locally
boolean
If true, then the Docker image won't be deleted on destroy operation. If this is false, it will delete the image from the docker local storage on destroy operation.
```

После изменения ключа `keep_locally = true` на `keep_locally = false` команда `terraform destroy` удалила и `docker container` и `docker image`.

![terraform destroy](img/ter-01_06.png)
