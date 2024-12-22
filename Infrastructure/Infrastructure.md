@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Deployment.puml

!include <tupadr3/devicons2/react_original>
!include <tupadr3/devicons2/nginx_original>
!include <tupadr3/devicons2/go>

title Диаграмма развертывания системы OutTrack на инфраструктуре клиента

System_Ext(calendar, "Календарь", "Личное расписание, дедлайны задач и время видеоконференций") 
System_Ext(mail, "Уведомления", "Оповещение о назначении и изменении задач, новых сообщениях и комментариях и начале конференций") 
System_Ext(video, "Видеовстреча", "Проведения видеоконференций")

Deployment_Node(vdc, "Виртуальный дата центр клиента"){
    
        Deployment_Node(webAppNode, "Нода веб-приложения", "Отдельная виртуальная машина"){
            Deployment_Node(k3s-pod-web-app-1, "Под веб-приложения"){
                Container(webAppPod, "Web App", "web-app-pod-1", "Веб-приложение, через которое пользователи взаимодействуют с системой", $sprite="react_original")
            }
        

        Deployment_Node(gwNode, "Gateway Node", "Отдельная виртуальная машина"){
            Deployment_Node(gw-pod-1, "Под API Gateway", "gw-pod-1"){
                Container(gwPod, "API-gateway", "OpenResty, Nginx, Lua", "Единая точка вызова для API, контроль токенов авторизации, middleware конвеера запросов", $sprite="nginx_original")
            }
            Container(lb, "Load balancer", "klipper-lb", "Распределение нагрузки между репликами монолита, встроенный load balancer")
        }

        'Ноды монолита
        Deployment_Node(k3s-node-monolith-1, "Нода монолита", "Отдельная виртуальная машина"){
            Deployment_Node(k3s-pod-monolith-1, "Под монолита"){
                Container(monolithPod1, "Инстанс монолита", "monolith", "Предоставляет API, реализует бизнес логику", $sprite="go")
            }
            Deployment_Node(k3s-pod-slave-db-1, "Под slave базы данных"){
                ContainerDb(slaveDbPod1, "Slave DB", "MySQL", "Slave база данных для реплики монолита", "", $sprite="mysql")
            }
        }

        Deployment_Node(k3s-node-monolith-2, "Нода монолита", "Отдельная виртуальная машина"){
            Deployment_Node(k3s-pod-monolith-2, "Под монолита"){
                Container(monolithPod2, "Инстанс монолита", "monolith", "Предоставляет API, реализует бизнес логику", $sprite="go")
            }
            Deployment_Node(k3s-pod-slave-db-2, "Под slave базы данных"){
                ContainerDb(slaveDbPod2, "Slave DB", "MySQL", "Slave база данных для реплики монолита",  $sprite="mysql")
            }
        }

        Deployment_Node(masterDbNode, "Нода Master базы данных", "master-db-node", "Кластер состоит из одного хоста БД — виртуальной машины с развернутой СУБД"){
            Deployment_Node(k3s-pod-master-db, "Под Master базы данных", "master-db-pod", "Кластер состоит из одного хоста БД — виртуальной машины с развернутой СУБД"){
                ContainerDb(masterDbPod, "Master DB", "MySQL", "Сохраняет состояние системы, которое синхронизируется со всеми slave нодами", "", $sprite="mysql")
            }
        }
    }
}

webAppPod ----> gwPod
gwPod ----> lb
lb ----> monolithPod1
lb ----> monolithPod2
monolithPod1 ----> masterDbPod
monolithPod2 ----> masterDbPod
masterDbPod ----> slaveDbPod1
masterDbPod ----> slaveDbPod2
monolithPod1 ----> video
monolithPod1 ----> calendar
monolithPod1 ----> mail
monolithPod2 ----> video
monolithPod2 ----> calendar
monolithPod2 ----> mail
SHOW_LEGEND()
@enduml