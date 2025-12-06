# Автоматизация сборки и деплоя Python Docker-приложений через Jenkins с интеграцией GitHub
***

## Введение

Цель данной лабораторной работы — освоить практические навыки автоматизации процесса непрерывной интеграции и развертывания (CI/CD) с использованием Jenkins — мощного инструмента автоматизации сборки, а также наладить взаимодействие с удалённым репозиторием GitHub посредством webhook, обеспечивающим мгновенный запуск сборки при каждом изменении кода.

***
## 1 этап

# Зашёл на сайт по своему логину и паролю
<img width="1837" height="928" alt="2025-12-06_13-41-23" src="https://github.com/user-attachments/assets/cba62c5c-74b3-47f2-8ff2-ad067fd4e5b6" />

# И создал сборку pipeline 
<img width="1837" height="925" alt="2025-12-06_13-42-12" src="https://github.com/user-attachments/assets/d7b4305d-53d8-404f-b21c-911b29760fa7" />

# В область Script добавил код из методички, изменив необходимые значения переменных
```
pipeline {
    agent any
    
    parameters {
        string(name: 'STUDENT_NAME', defaultValue: 'Ravil', description: 'Имя студента')
        string(name: 'PORT', defaultValue: '8011', description: 'Порт')
    }
    
    stages {
        stage('Удаляем старые контейнеры и образы') {
            steps {
                script {
                    // Останавливаем и удаляем контейнер с нужным именем, если он есть
                    sh "docker ps -a -q --filter name=hello-zrr-container | xargs -r docker rm -f"
                    // Удаляем образ с нужным именем, если он есть
                    sh "docker images -q student-zrr-app | xargs -r docker rmi -f"
                }
            }
        }
        stage('Выгружаем код из репозитория') {
            steps {
                git branch: 'main', url: 'https://github.com/tasher2112/hellojenkins.git'
            }
        }
        stage('Собираем docker image') {
            steps {
                script {
                    dockerImage = docker.build("student-zrr-app")
                }
            }
        }
        stage('Запускаем тесты в докере') {
            steps {
                script {
                    dockerImage.inside {
                        sh 'python -m unittest test_app.py'
                    }
                }
            }
        }
        stage('Запускаем докер контейнер') {
            steps {
                script {
                    sh "docker run -d --name hello-zrr-container -p ${params.PORT}:${params.PORT} -e STUDENT_NAME='${params.STUDENT_NAME}' -e PORT=${params.PORT} student-zrr-app"
                }
            }
        }
    }
} 
```

## После этого создал вебхук внутри гитхаба, для того чтобы коммиты из ветки автоматически отправлялись на сайт, и он пересобирал сборку.

<img width="1356" height="925" alt="2025-12-06_13-49-39" src="https://github.com/user-attachments/assets/c89b973a-3047-4b3d-ba75-57d9ae6e39d1" />

## После некоторых неудачных использований вебхука поставил флажок в дженкинсе на триггер по вебхуку
<img width="985" height="579" alt="2025-12-06_13-53-02" src="https://github.com/user-attachments/assets/b5e67126-b7e3-4f95-a7d8-2e71b546dc44" />
<img width="1832" height="932" alt="2025-12-06_13-55-00" src="https://github.com/user-attachments/assets/e540e0a2-b74b-4e2a-af40-72ba5575ea6b" />


***
