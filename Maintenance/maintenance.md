@startuml
!theme vibrant

actor "Разработчик" as developer
package "Учет задач" as task_record{
    component "Avion"
    component "Jira"
    developer --> [Jira] : "Выбрать задачу для разработки"
    [Avion] -> [Jira] : "Нарезка задач"
}


package "Разработка и развертывание" as development {
    [GitHub] as vcs << (C, #FFAAAA) >>
    [GitHub Actions (main)] as cicd_prod << (C, #FFAAAA) >>
    [GitHub Actions (develop)] as cicd_dev << (C, #FFAAAA) >>

    developer --> vcs : "Push изменений"
    vcs --> cicd_prod : "Триггер CI/CD (main)"
    vcs --> cicd_dev : "Триггер CI/CD (develop)"
    
    package "CI/CD Pipeline (PROD)" as build_prod {
      [Build & Test] as build_test_prod << (C, #FFAAAA) >>
      [Security Scan] as security_scan_prod << (C, #FFAAAA) >>
      [Deploy to Prod] as deploy_prod << (C, #FFAAAA) >>
    }
    
    package "CI/CD Pipeline (DEV)" as build_dev {
      [Build & Test] as build_test_dev << (C, #FFAAAA) >>
      [Security Scan] as security_scan_dev << (C, #FFAAAA) >>
      [Deploy to Dev] as deploy_dev << (C, #FFAAAA) >>
    }

    [ServerProd] as container_registry_prod << (C, #FFAAAA) >>
    [ServerQa] as container_registry_qa << (C, #FFAAAA) >>

    ' Pipeline PROD
    cicd_prod --> build_test_prod : "Запуск пайплайна"
    build_test_prod --> security_scan_prod : "Проверка безопасности"
    security_scan_prod --> deploy_prod : "Развертывание"
    deploy_prod --> container_registry_prod : "Push образов"

    ' Pipeline DEV
    cicd_dev --> build_test_dev : "Запуск пайплайна"
    build_test_dev --> security_scan_dev : "Проверка безопасности"
    security_scan_dev --> deploy_dev : "Развертывание"
    deploy_dev --> container_registry_qa : "Push образов"
}

package "Яндекс.Облако Kubernetes - QA" as k8s_qa {
  [Docker] as docker_qa << (C, #FFAAAA) >>
  [Containers] as containers_qa << (C, #FFAAAA) >>
  docker_qa --> containers_qa : "Рарзвертывание контейнеров"

  container_registry_qa --> docker_qa : "Pull образов"
}

package "Яндекс.Облако Kubernetes - Prod" as k8s_prod {
  [Docker] as docker_prod << (C, #FFAAAA) >>
  [Containers] as containers_prod << (C, #FFAAAA) >>
  docker_prod --> containers_prod : "Рарзвертывание контейнеров"

  container_registry_prod --> docker_prod : "Pull образов"
}
    
package "Система управления задачами" as maintenance {
  actor "DevOps" as devops
  
  package "Яндекс.Облако" as cloud {
      [Система мониторинга Zabbix] as monitoring_service << (C, #FFAAAA) >>
      [Хранилище резервных копий] as backupsStorage << (C, #FFAAAA) >>
  }

  devops --> backupsStorage : "Управляет бекапами БД"
  devops --> monitoring_service : "Мониторит ошибки"
  developer --> monitoring_service : "Мониторит ошибки"
  monitoring_service --> containers_prod : "Опрашивает приложение и проверяет работоспособность"
}

package "Внешние сервисы" as outer_services {
    package "Канал уведомлений" as notifications {
        [Почтовый шлюз] as email_service << (C, #FFAAAA) >>
    }
}
email_service --> devops : "уведомление"
monitoring_service --> email_service : "Отправка уведомлений об ошибках и предупреждения"

package "Тестирование" as qa {
  actor "Тестировщик" as qaEngineer

  [GitHub] as vcsQa << (C, #FFAAAA) >>
  [GitHub Actions (main)] as cicd_tests << (C, #FFAAAA) >>

  ' Связи автотестов
  qaEngineer --> vcsQa : "Push изменений"
  vcsQa --> cicd_tests : "Триггер CI/CD (Автотесты)"

  package "CI/CD Pipeline" as build_pipeline_tests {
    [Build] as build_tests << (C, #FFAAAA) >>
    [Security Scan] as security_scan_tests << (C, #FFAAAA) >>
    [Deploy to Dev] as deploy_tests << (C, #FFAAAA) >>
  }

  [AutotestsServer] as container_registry_tests << (C, #FFAAAA) >>

  cicd_tests --> build_tests : "Запуск пайплайна"
  build_tests --> security_scan_tests : "Проверка безопасности"
  security_scan_tests --> deploy_tests : "Развертывание"
  deploy_tests --> container_registry_tests : "Push образов"

  container_registry_tests --> containers_qa : "Автоматическое тестирование"
  qaEngineer --> containers_qa : "Ручное тестирование"
}

@enduml