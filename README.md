# PHPBase
# Основы PHP

## Домашнее задание № 9.

Скорректируйте список пользователей так, чтобы все пользователи с правами администратора в таблице видели две дополнительные ссылки – редактирование и удаление пользователя. При этом редактирование будет переходить на форму, а удаление в асинхронном режиме будет удалять пользователя как из таблицы, так и из БД.

### Решение:

Добавим в файл user-index.tpl 2 ссылки на редактирование и удаление с проверкой на админа:

```
{% if isAdmin %}
  <th scope="col">Редактирование</th>
  <th scope="col">Удаление</th>
{% endif %}
```

```
{% if isAdmin %}
  <td scope="col"><a href="/user/edit/?id_user={{ user.getUserId }}">Редактирование</a></td>>
  <td scope="col"><a href="/user/delete/?id_user={{ user.getUserId }}">Удаление</a></td>>
{% endif %}
```

Также добавим функцию удаления с POST запросом:

```
function delete(userId) {
  $.ajax({
    method: 'POST',
    url: "/user/delete/",
    data: { id: userId}
  }).done(function (response) {
    document.getElementById("userId" + userId).remove();
  });
}
```

Редактирование ведет на форму создания пользователя user-form.tpl

Создадим метод для удаления пользователя в классе UserController (для удаления пользователя в асинхронном режиме используем метод POST):

```
public function actionDelete(): string {
  if(User::exists($_POST['id'])) {
    User::deleteFromStorage($_POST['id']);
    return $this->actionIndex();
  }
  else {
    throw new Exception("Пользователь не существует");
  }
}
```

В классе User создаём недостающие методы deleteFromStorage и exists:

```
public static function deleteFromStorage(int $user_id) : void {
  $sql = "DELETE FROM users WHERE id_user = :id_user";

  $handler = Application::$storage->get()->prepare($sql);
  $handler->execute(['id_user' => $user_id]);
}

public static function exists(int $id): bool{
  $sql = "SELECT count(id_user) as user_count FROM users WHERE id_user = :id_user";

  $handler = Application::$storage->get()->prepare($sql);
  $handler->execute([
    'id_user' => $id
  ]);

  $result = $handler->fetchAll();

  if(count($result) > 0 && $result[0]['user_count'] > 0){
    return true;
  }
  else{
    return false;
  }
}
```

Не забываем, что удаление пользователя может делать только администратор, поэтому добавляем в классе UserController в массив actionsPermissions сточку: 

```
'actionDelete' => ['admin']
```

Итоговый массив:

```
protected array $actionsPermissions = [
  'actionHash' => ['admin', 'some'],
  'actionSave' => ['admin'],
  'actionDelete' => ['admin']
];
```
