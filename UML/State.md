@startuml
[*]     --> ProjectCreated :Создать проект
ProjectCreated : Проект создан
ProjectCreated --> TaskCreated : Создать задачу
TaskCreated : Задача создана
TaskCreated --> NotificationSent: Назначить исполнителя
NotificationSent: Исполнитель получил уведомление
NotificationSent --> StartExecution : Начать задачу
StartExecution : Исполнитель начал выполнение задачи
StartExecution --> TimeTracked : Начать отсчет времени
TimeTracked: Время выполнения сохранено
TimeTracked --> CalendarUpdated: Загрузить в календарь
CalendarUpdated: Сроки задачи добавлены в календарь
CalendarUpdated --> TaskCompleted: Завершить задачу
TaskCompleted: Задача завершена
TaskCompleted --> ReportSended: Отправить отчёт
ReportSended: Отчёт отправлен
ReportSended --> TaskTested: Принять проверенную задачу
TaskTested: Задача проверена
TaskTested --> TaskArchived: Архивировать задачу
TaskArchived: Задача архивирована
TaskArchived --> TaskDeleted: Удалить задачу
TaskDeleted: Задача удалена
TaskDeleted --> [*]
@enduml