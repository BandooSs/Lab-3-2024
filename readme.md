# Лабораторная работа №3

## Airflow и MLflow - логгирование экспериментов и версионирование моделей

В рамках данной лабораторной работы предлагается построить два пайплайна:

1. Пайплайн, который обучает любой классификатор из sklearn по заданному набору параметров.
2. Пайплайн, который выбирает лучшую модель из обученных и производит её хостинг.

Для построения такого пайплайна воспользуемся следующими инструментами:

- Apache Airflow
- MLflow

## Подготовка к выполнению задания

Для выполнения лабораторной работы рекомендуется использовать докер контейнеры из подготовительного репозитория: https://github.com/ssau-data-engineering/Prerequisites/tree/main

## Задание на лабораторную работу

### Пайплайн для обучения классификаторов

Построенный пайплайн будет выполнять следующие действия поочередно:

1. Производить мониторинг целевой папки на предмет появления новых конфигурационных файлов классификаторов (в форматах `.json` или `.yaml`).
2. Обучать классификатор в соответствии с полученными параметрами. 
   - Производить логгирование параметров модели в **MLflow**. 
   - Производить логгирование процесса обучения **MLflow**. 
   - Производить тестирование модели и сохранять его результаты в **MLflow**.
3. Сохранять обученный классификатор в `model registry` **MLflow**.

### Пайплайн для хостинга лучшей модейли

Построенный пайплайн будет выполнять следующие действия поочередно:

1. В соответствии с таймером производит валидацию новых моделей из `model registry`.
2. Модель с лучшим показателем метрики переводится на `stage: Production`
3. (Опционально) произвести хостинг лучшей модели

## Сдача лабораторной работы

Для успешной сдачи лабораторной работы ваш репозиторий должен содержать следующее:

1. Обучить при помощи пайплайнов не менее пяти классификаторов с различными параметрами
2. Отчет, описывающий этапы выполнения работы (скриншоты, описание встреченных проблем и их решения приветствуются) в формате .pdf или .md.
   - *Отчет также должен содержать скриншоты со страниц как Airflow, так и MLflow*
3. Программный код, реализующий пайплайны.
4. Открыт Pull Request в данный репозиторий.

## FAQ

- В рамках настоящей работы можно обойтись без использование `DockerOperator`
- Все сохраняемые в рамках работы модели - сохраняются в объектное хранилище **Minio** (доступное по адресу: http://localhost:9000 minio@minio123).
Это можно использовать при построении сложных пайплайнов для осуществления обмена данными между различными узлами системы.
- Чтобы Airflow смог сохранить модель - необходимо в файле описывающем **DAG** установить также переменные среды:
    ```python
    import os

    os.environ["AWS_ACCESS_KEY_ID"] = "minio"
    os.environ["AWS_SECRET_ACCESS_KEY"] = "minio123"
    os.environ["MLFLOW_S3_ENDPOINT_URL"] = "http://minio:9000"
    ```
- Пример файла конфигурации классификатора:
    ```yaml
    classificator: sklearn.neural_network.MLPClassifier
    kwargs:
    - hidden_layer_sizes:
        - 20
        - 100
        - 20
    - activation: relu
    - solver: adam
    - learning_rage: 0.001
    ```
- При выполнении работы настоятельно рекомендуется пользоваться официальной докумментацией:
  * https://mlflow.org/docs/latest/model-registry.html#mlflow-model-registry
  * https://mlflow.org/docs/latest/tracking.html#logging-data-to-runs
  * https://mlflow.org/docs/latest/models.html
  * https://mlflow.org/docs/latest/search-runs.html
 
#  Курицын Никита 6232
# Task_1 Пайплайн, который обучает любой классификатор из sklearn по заданному набору параметров.
Был выбран датасет [smoke-detection-dataset](https://www.kaggle.com/datasets/deepcontractor/smoke-detection-dataset)

    1. Был реализован пайпайн  wait_for_new_json_file >> train_models. Граф пайпланйна представлен ниже.  
![Пример изображения](https://github.com/BandooSs/Lab-3-2024/blob/main/data/LR_3_task1/airflow_graph.png)
    
    2. Был составлен json файл с информацией о классификаторах, а также с информацией о датасете.
    3. PythonSensor(wait_for_new_json_file) - сканер файла, искал появление файла json.
    3. После появления файла активировался BashOperator(train_models) для получения информации из файла json, подготовки датасета, обучения классификаторов, отрпавления моделей на mlflow моделей(отрпавлялись значения метрик на выборках, время обучения, параметры модели).
![Пример метрик](https://github.com/BandooSs/Lab-3-2024/blob/main/data/LR_3_task1/mlflow_metrics.png)

# Task_2 Пайплайн, который выбирает лучшую модель из обученных и производит её хостинг.
    1. Был реализован пайплайн get_best_model.
    2. PythonOperator(get_best_model) - вызывает функцию get_best_model(), которая считывает валидационную выборку, модели из mlflow.
    3. Выбирает наилучшую модель по точности на валидационной выборке и переводит ее в статус 'Production'.
![Пример метрик](https://github.com/BandooSs/Lab-3-2024/blob/main/data/LR_3_task1/mlflow_models.png)
