# Каталог программной реализации нейросети для предотвращения сетевых угроз типа Кохонена, тестовые данные и демонстрации работы нейросети команды CVA Technologies

Ссылка на поэтапное выполнение ПО в рамках енвайромента среды Jupyter Python с частью обучающих и тестовых данных - при работе с входными файлами:

https://github.com/avgorinych/CVA_IDS/blob/master/ml/notebooks/03_ml-prototype/ml-prototype_cva.ipynb

Работа в потоковом режиме на данный момент не предусмотрена - будет реализована в дальнейшем!

Использованные библиотеки:

  - click=7.0=py37_0
  - catboost=0.18.1=py37_0
  - cloudpickle=1.2.2=py_0
  - imbalanced-learn=0.5.0=py_0
  - matplotlib=3.1.1=py37_1
  - mypy=0.750
  - numpy=1.17.2=py37h95a1406_0
  - pandas=0.25.2=py37hb3f55d8_0
  - pip=19.2.3=py37_0
  - pylint=2.4.4
  - pytest=5.2.1=py37_0
  - pytest-runner=5.1=py_0
  - python=3.7.3=h33d41f4_1
  - python-dateutil<2.8.1
  - requests<2.21.0
  - scikit-learn=0.21.3=py37hcdab131_0
  - scipy=1.3.1=py37h921218d_2
  - seaborn=0.9.0=py_1
  - setuptools=41.6.0=py37_1
  - pip:
    - mlflow==1.4
    - sagemaker==1.44.3
    - h5py==2.10.0
    - hyperopt==0.2.2
    - keras==2.3.1
    - keras-applications==1.0.8
    - keras-preprocessing==1.1.0
    - tables==3.6.1
    - tensorflow-estimator==2.0.0
    - tensorflow-gpu==2.0.0 

В данный момент тестовый способ проверки - внутри исполняющей системы с испольнением Python кода:

https://github.com/avgorinych/CVA_IDS/blob/master/ml/notebooks/03_ml-prototype/ml-prototype_cva.ipynb

Реализация запуска на облаке AWS на этапе разработки!

Установка конда:

```
conda env create -f environment.yml
```

```
conda activate ml-ids

pip install -e .
```

Создание датасета:

```
make split_dataset \
  DATASET_PATH={path-to-source-dataset}
```

Команда прочитает исходный датасет и  преобразует его в несколько train/validation/test сетов в пересчете 80%/10%/10%. Датасет исходный должен содержать каталог с  `.csv` файлами.
Мы используем датасет [CIC-IDS-2018 dataset](https://www.unb.ca/cic/datasets/ids-2018.html) .

Тренировка локальной модели:

```
make train_local
```

Для расширенных параметров (если датасет в другом месте):

```
make train_local \
  TRAIN_PATH={path-to-train-dataset} \
  VAL_PATH={path-to-train-dataset} \
  TEST_PATH={path-to-train-dataset} \
  TRAIN_PARAM_PATH={path-to-param-file}
```

Для разворачивания с помощью MLflow CLI:

```
mlflow models serve -m build/models/gradient_boost -p 5000
```

Разворачивание с помощью докера:

```
mlflow models build-docker -m build/models/gradient_boost -n ml-ids-classifier:1.0

docker run -p 5001:8080 ml-ids-classifier:1.0
```

Для тренировки модели в облаке Amazon SageMaker:

```
# build a new docker container for model training
make sagemaker_build_image \
  TAG=1.0

# upload the container to AWS ECR
make sagemaker_push_image \
  TAG=1.0

# execute the training container on Amazon SageMaker
make sagemaker_train_aws \
  SAGEMAKER_IMAGE_NAME={ecr-image-name}:1.0 \
  JOB_ID=ml-ids-job-0001
```

Требуется AWS аккаунт с [AWS CLI](https://aws.amazon.com/cli/), [AWS ECR](https://aws.amazon.com/ecr/) .