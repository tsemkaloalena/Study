**Конфигурация пользователя**<br>
git config user.name<br>
git config user.email записывается в .git/config<br>
**3 уровня конфигурации:**<br>
локальный --local (default) (<project>/.git/config), глобальный --global (~/.gitconfig, C:/users/<user>/.gitconfig) и системный --system (C/:Program Files/Git/etc/gitconfig)<br>
**Чтение конфигураций:**<br>
git config --list <br>
git config --list --global<br>
**Удалить секцию**<br>
git config --remove-section user<br>
**Добавление своих команд**<br>
git config alias.sayhi ‘echo …’<br>
Вызов git sayhi<br>
**Просмотр информации**<br>
git config -h<br>
Более подробно git help config<br>
Внутри: <br>
 ⁃ поиск строки - “/<строка>”<br>
 ⁃ n - поиск вперёд<br>
 ⁃ Shift+n - поиск назад<br>
<br>
**Создание репозитория:**<br>
git init <br>
git status - просмотр состояния репозитория <br>
git add index.html - добавление файла в индекс (отслеживание)<br>
git commit<br>
git show - информация о коммите <br>
Гит не видит пустые папки<br>
git commit -a (--all) - коммитит все изменения<br>
git commit -m ‘’ index.html - коммитит указанный файл<br>
git rm index.html - удаление файла<br>
<br>
**Переключение в другую ветку**<br>
git checkout …<br>
git branch - создание ветки<br>
git branch -v - текущее состояние ветки<br>
**Сохранение незакоммиченных файлов при переключении веток**<br>
git stash<br>
git stash pop<br>
**Переместить указатель ветки на конкретный коммит**<br>
git branch -f master commit-id<br>
**Detached HEAD – состояние, когда мы не находимся на какой то определенной ветке**<br>
git log – информация о всех коммитах в ветки репозитория от HEAD<br>
git show filename.html – показать содержимое файла из HEAD<br>
git show :/MIB-1488 – показать коммит с MIB-1488 в описании<br>
**Объединение веток**<br>
git merge fix – объединить текущую ветку с fix<br>
в файле .git/ORIG_HEAD находится название бывшего HEAD текущей ветки<br>
**Удаление ветки**<br>
git branch -d fix – удалить ветку, все коммиты которой есть в другой ветке<br>
git branch -D fix – удалить ветку со всеми её личными ком митами<br>
удалением можно отменить созданием git branch fix commit-id<br>
**Логи по всем переключениям HEAD**<br>
git reflog –date=iso<br>
можно восстановить ветку по старым коммитам git branch fix ‘HEAD{number from reflog}’ <br>
git checkout @{-1} – переключение на предыдущую ветку<br>
**Полностью очистить изменения**<br>
git reset –hard – удалит все отслеживаемые файлы, которые появились с момента прошлого коммита<br>
git clean -dxf – удалит все неостлеживаемые, игнорируемые файлы и директории, которые появились с момента прошлого коммита<br>
@- head<br>
~ - родитель текущего коммита<br>
**Сброс до коммита**<br>
git reset –hard commit_id<br>
если нужно вернуться к удаленному, то git reset –hard ORIG_HEAD (буквально ORIG_HEAD, это ссылка на удаленный коммит)<br>
git reset –soft @~ отменяет коммит, но оставляет в индексе изменения сделанные этим коммитом<br>
git commit –amend - объединение отката до прошлого коммита плюс коммит новых изменений<br>
git reset = git reset –mixed – откат до коммита указанного с удалением файлов из индексирования<br>
**История репозитория**<br>
git diff – различия между файлами в коммитах и ветках<br>
git diff –cached – проиндексированные изменения, которые попадут в следующий коммит<br>
**Логирование может быть форматировано флагами**<br>
git log –pretty=format:’format_style’<br>
git log –grep Run branch_name – поиск коммитов с описанием содержащим Run из ветки branch_name<br>
git blame – показывает в каком последнем коммите была изменена каждая из строк файла<br>
**Объединение веток**<br>
git merge feature – объединит ветку feature с текущей веткой<br>
git log master –oneline –first-parent – показывает коммиты только от ветки мастер<br>
git merge –squash feature отдельным коммитом в ветку текущую будут добавлены все изменения между текущей и feature<br>
**Отмена merge**<br>
git reset –hard @~<br>
**Копирование коммитов**<br>
git cherry-pick commit_id – скопирует коммит в текущую ветку<br>
git cherry-pick -n commit_id – скопирует изменения в индекс, но не будет коммитить<br>
**Перемещение коммитов**<br>
git rebase master – скопировать все коммиты текущей ветки поверх ветки master<br>
 Операция merge безопасна, она не меняет предыдущие. Операция rebase может привести к проблемам из-за перекопированния. Rebase можно делать локально и если веткой занимается один человек.<br>
git rebase -x ‘command’ – выполнение комманд после каждого копирования коммита, используется в тестах и позволяет выявить ошибки<br>
git rebase –onto master feature – переместить все коммиты текущей ветки на мастер начиная от отхождения от feature<br>
![image](https://github.com/user-attachments/assets/4bdcbb6b-56ea-4cd9-aa7d-3aa8f3003128)<br>
<br>
git rebase -i master – интерактивное перебазирование, открывает редактор с возможностью выбора действий<br>
![image](https://github.com/user-attachments/assets/305f79f8-308b-430a-b759-06fdd5e45f06)<br>
<br>
git rebase -i @~count – открыть редактор для count предыдущих коммитов текущей ветки<br>
**Отмена коммитов**<br>
git revert – @ - создать коммит, удаляющие изменения из последнего коммита<br>
В сценарии где произошел revert коммита merge, ту ветку которая мерджилась стоит rebase поверх revert<br>
git rebase –onto master commit_id feature, где commit_id это коммит родитель ветки feature<br>
можно также создать полную копию ветки, которая однако не будет иметь связей с master, а от этого и не будет пропущенных коммитов <br>
git rebase commit_id –no-ff. Это не просто подвинет указатель на текущую ветку, а буквально перекопирует коммиты и только потом передвинет указатель<br>
