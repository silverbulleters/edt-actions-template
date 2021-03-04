# edt-actions-template

Шаблон нового проекта EDT для автоматизации на GitHub Actions.

## Ограничения

Работает на собственном [github runner](https://docs.github.com/en/free-pro-team@latest/actions/hosting-your-own-runners). 
Предварительно его нужно настроить. Устанавливаем:
* Платформу 1С нужной версии. Например 8.3.17.1846
* EDT 2020.6.2 или новее
* OneScript 1.4
* Пакеты OneScript:
    - vanessa-runner

## Автоматизация

Автоматизированы процессы:

* Конвертация конфигурации и расширения с тестами в формат xml
* Cинтаксический контроль
* Модульное тестирование (через Vanessa Automation)
* Статический анализ SonarQube
* Публикация отчета о тестировании Allure
* Публикация CF-файла в релизах

## Секреты

Перед началом запуска `workflow` нужно настроить секреты в проекте (Settings -> Secrets). Нужны следующие переменные:
* SONARQUBE_HOST - ссылка на сервис SonarQube. Например: https://open.checkbsl.org/.
* SONARQUBE_TOKEN - токен доступа на сервис SonarQube. Генерируется на странице профиля в SonarQube.

### Структура шаблона

```
├── .github                                 # Вспомогательный каталог github
|   └── workflows                           # Каталог с "работами" для GitHub.Actions
├── src                                     # Каталог для исходных кодов
|   ├── ProjectEDT                          # Шаблонный каталог проекта с конфигурацией
|   └── ProjectEDT.Tests                    # Шаблонный каталог расширения для тестов
├── tools                                   # Вспомогательный каталог для инструментов и настроек
|    ├── json                               # Каталог с конфигурационными файлами json
|    └── vanessa-automation-single.epf      # Singlton версия Vanessa.Automation
├── sonar-project.properties                # Конфигурационный файл для анализа в SonarQube
```

## Блок CI

Блок CI описан в файле [ci.yml](/.github/workflows/ci.yml). 

### Предварительная настройка

1. Изменить названия проектов `ProjectEDT` и `ProjectEDT.Tests` на свои.

* `ProjectEDT` - для проекта EDT с конфигурацией 1С.
* `ProjectEDT.Tests` - для расширения конфигурации с модульными тестами. Тесты написаны с помощью плагина [ru.capralow.dt.unit.launcher](https://github.com/DoublesunRUS/ru.capralow.dt.unit.launcher).

2. Условие `if: github.repository == 'silverbulleters/edt-actions-template'` в задании `sonar-analyze` меняем на условие с "путем" к проекту. 
Например: `if: github.repository == 'author/my-project`.

3. Адаптировать конфигурационные файлы [VRunner.json](tools/json/VRunner.json) и [VBParamsVA.json](tools/json/VBParamsVA.json) под себя. 
* Заменить `ProjectEDT.Tests` на название проекта с тестами.
* В первом приближении фреймворк Vanessa.Automation лежит в каталоге tools. При использовании другой версии, эту обработку нужно заменить.

### Действия

В workflow 3 задачи:
* `qa` - сборка и тестирования проекта с помощью vanessa-runner и Vanessa.Automation
* `sonar-analyze` - статический анализ проекта EDT
* `publish-allure` - публикация отчетов Allure на странице GitHub Pages. Пример адреса: https://silverbulleters.github.io/edt-actions-template

## Блок CD

Блок CD описан в файле [cd.yml](/.github/workflows/cd.yml). При создании нового релиза будет 
автоматически собрана cf конфигурации и опубликована на странице релиза.