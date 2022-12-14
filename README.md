# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил:
- Батраков Дмитрий Антонович
- НМТ212701
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.

## Задание 1
### 
Реализовать систему машинного обучения в связке Python - Google-Sheets – Unity. 
При выполнении задания можно использовать видео-материалы и исходные данные, предоставленные преподавателями курса.

В ходе работы:

-был созданн новый прокт в Unity

-был скачан и добавлен в проект ML агент

-были последовательно добавлены .json – файлы: 

      ml-agents-release_19 / com,unity.ml-agents / package.json 
  
      ml-agents-release_19 / com,unity.ml-agents.extensions / package.json
  
В вкладке Components внутри Unity появилась строка ML Agent.

-через Anaconda Prompt был создан и активирован ML Agent
```py
(base) C:\Windows\system32>conda create -n MlAgent python=3.6.13
```
```py
C:\Windows\system32>conda activate MLAgent
```
-скачаны необходимые библиотеки: mlagents 0.28.0 и torch 1.7.1
```py
(MLAgent) C:\Windows\system32>pip install mlagents==0.28.0
```
```py
(MLAgent) C:\Windows\system32>pip install torch~=1.7.1 -f https://download.pytorch.org/whl/torch_stable.html
```
-на сцене были созданы плоскость, куб и сфера так, как было представленно в методических материалах
![image](https://user-images.githubusercontent.com/113825126/197381204-a0219060-3168-44fc-a9e0-529658ed22ba.png)

-был создан простой C# скрипт-файл, он был подключен к сфере

-в скрипт-файл RollerAgent.cs был добавлен код, опубликованный в материалах лабораторных работ
```py
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if(distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}

```

-объекту «сфера» были добавлены компоненты Rigidbody, Decision Requester, Behavior Parameters

-они были настройены так, как показано в методичеких материалах

![image](https://user-images.githubusercontent.com/113825126/197381495-fa270491-28d6-4544-92ed-e9862a9460f8.png)

-в корень проекта был добавлен файл конфигурации нейронной сети
```py
behaviors:
  RollerBall:
    trainer_type: ppo
    hyperparameters:
      batch_size: 10
      buffer_size: 100
      learning_rate: 3.0e-4
      beta: 5.0e-4
      epsilon: 0.2
      lambd: 0.99
      num_epoch: 3
      learning_rate_schedule: linear
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 500000
    time_horizon: 64
    summary_freq: 10000
 ```
 -была запущена работа ml-агента
 ```py
 (MLAgent) C:\Users\bda00\Documents\GitHub\lab1-Unity-\MLATest>mlagents-learn rollerball_config.yaml --run-id=RollerBall --force
 ```
 
 -была запущена сцена в проекте Unity, была проверена работа ml-агента
 
 -были сделаны 3, 9, 27 копий модели «Плоскость-Сфера-Куб», была запущена симуляция сцены, произведены наблюдения за результатом обучения модели
 ![image](https://user-images.githubusercontent.com/113825126/197382730-5eae7694-3cbe-4da0-b90e-1fd0c0542300.png)
 
-после завершения обуучения была проверена работа модели
![image](https://user-images.githubusercontent.com/113825126/197382845-1e7329be-8686-4654-aa1c-e1ddae0f7f4f.png)

-сделаны выводы, они представлены в разделе "Выводы"

## Задание 2
### Подробно опишите каждую строку файла конфигурации нейронной сети, доступного в папке с файлами проекта по ссылке. Самостоятельно найти информацию о компонентах Decision Requester, Behavior Parameters, добавленных на сфере.

Содержимое файла конфигурации:
 ```py
behaviors:
  RollerBall:
    trainer_type: ppo
    hyperparameters:
      batch_size: 10
      buffer_size: 100
      learning_rate: 3.0e-4
      beta: 5.0e-4
      epsilon: 0.2
      lambd: 0.99
      num_epoch: 3
      learning_rate_schedule: linear
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 500000
    time_horizon: 64
    summary_freq: 10000
 ```
 trainer_type: ppo – тип используемого тренажёра


 ```batch_size: 10 ``` - Количество опытов в каждой итерации градиентного спуска. Это всегда должно быть в несколько раз меньше, чем размер буфера. Если использовать непрерывные действия, это значение должно быть большим (порядка 1000 с). Если использщвать только дискретные действия, это значение должно быть меньше (порядка 10 с).

 ```buffer_size: 100 ``` -  для PPO: количество опытов, которые необходимо собрать перед обновлением модели политики. Соответствует тому, сколько опыта должно быть собрано, прежде чем мы будем изучать или обновлять модель. Это значение должно быть в несколько раз больше, чем buffer_size. Обычно больший размер буфера соответствует более стабильным обновлениям обучения. (по умолчанию 10240 для PPO)

 ```earning_rate: 3.0e-4 ``` - Начальная скорость обучения для градиентного спуска. Соответствует силе каждого шага обновления градиентного спуска. Обычно это значение следует уменьшать, если обучение нестабильно, а вознаграждение не увеличивается постоянно. (по умолчанию 3.0e-4)

 ```beta: 5.0e-4 ``` - Сила регуляризации энтропии, которая делает политику «более случайной». Это гарантирует, что агенты должным образом исследуют пространство действия во время обучения. Увеличение этого параметра обеспечит выполнение большего количества случайных действий. Это должно быть скорректировано таким образом, чтобы энтропия медленно уменьшалась вместе с увеличением вознаграждения. Если энтропия падает слишком быстро, рекомендуется увеличить бета, если энтропия падает слишком медленно - уменьшить. (по умолчанию 5.0e-3)

 ```epsilon: 0.2 ``` - Влияет на скорость изменения политики во время обучения. Соответствует допустимому порогу расхождения между старой и новой политикой при обновлении градиентного спуска. Установка небольшого значения этого параметра приведет к более стабильным обновлениям, но также замедлит процесс обучения. (по умолчанию 0,2)

 ```lambd: 0.99 ```  -  Параметр регуляризации, используемый при расчете обобщенной оценки преимущества (GAE). Это можно рассматривать как то, насколько агент полагается на свою текущую оценку стоимости при вычислении обновленной оценки стоимости. Низкие значения соответствуют большему полаганию на текущую оценку ценности (что может быть высоким смещением), а высокие значения соответствуют большему полаганию на фактические вознаграждения, полученные в среде (что может быть высокой дисперсией). Параметр обеспечивает компромисс между ними, и правильное значение может привести к более стабильному процессу обучения.(по умолчанию 0,95)

 ```num_epoch: 3 ``` - Количество проходов через буфер опыта при оптимизации градиентного спуска. Чем больше размер партии, тем больше это допустимо. Уменьшение этого параметра обеспечит более стабильные обновления за счет более медленного обучения. (по умолчанию 3)

 ```learning_rate_schedule: linear ``` - Определяет, как скорость обучения изменяется с течением времени. Для PPO рекомендуется снижать скорость обучения до значения max_steps, чтобы обучение сходилось более стабильно. Однако в некоторых случаях (например, при обучении в течение неизвестного времени) эту функцию можно отключить. (по умолчанию линейный для PPO)
Линейно затухает Learning_rate, достигая 0 при max_steps, в то время как Constant сохраняет скорость обучения постоянной для всего тренировочного прогона.

 ```normalize: false ``` - Применяется ли нормализация к входным данным векторного наблюдения. Эта нормализация основана на скользящем среднем и дисперсии векторного наблюдения. Нормализация может быть полезна в случаях со сложными задачами непрерывного управления, но может быть вредна для более простых задач дискретного управления. (по умолчанию false)

 ```hidden_units: 128 ``` - Количество юнитов в скрытых слоях нейронной сети. Соответствуют количеству единиц в каждом полносвязном слое нейронной сети. Для простых задач, где правильное действие представляет собой простую комбинацию входных данных наблюдения, это значение должно быть небольшим. Для задач, где действие представляет собой очень сложное взаимодействие между переменными наблюдения, это значение должно быть больше. (по умолчанию 128)

 ```num_layers: 2 ``` - Количество скрытых слоев в нейронной сети. Соответствует количеству скрытых слоев после ввода наблюдения или после кодирования CNN визуального наблюдения. Для простых задач меньше слоев, скорее всего, будут обучать быстрее и эффективнее. Для более сложных задач управления может потребоваться больше слоев. (по умолчанию 2)

 ```gamma: 0.99 ``` - Фактор скидки для будущих вознаграждений, поступающих из окружающей среды. Это можно рассматривать как то, как далеко в будущем агент должен заботиться о возможных вознаграждениях. В ситуациях, когда агент должен действовать в настоящем, чтобы подготовиться к вознаграждению в отдаленном будущем, это значение должно быть большим. В случаях, когда вознаграждение является более немедленным, оно может быть меньше. Должно быть строго меньше 1. (по умолчанию 0,99)

 ```py strength: 1.0 ``` - Коэффициент, на который умножается вознаграждение, данное средой. Типичные диапазоны будут варьироваться в зависимости от сигнала вознаграждения. (по умолчанию 1.0)


 ```max_steps: 500000 ``` - Общее количество шагов (т. е. собранных наблюдений и предпринятых действий), которые необходимо выполнить в среде (или во всех средах при параллельном использовании нескольких) перед завершением процесса обучения. Если в среде есть несколько агентов с одинаковым именем поведения, все шаги, предпринятые этими агентами, будут учитываться в одном и том же счетчике max_steps. (по умолчанию 500000)

 ```time_horizon: 64 ``` - Сколько шагов опыта нужно собрать для каждого агента, прежде чем добавить его в буфер опыта. Когда этот предел достигается до конца эпизода, оценка значения используется для прогнозирования общего ожидаемого вознаграждения из текущего состояния агента. (по умолчанию 64)

 ```summary_freq: 10000 ``` - Количество опытов, которое необходимо собрать перед созданием и отображением статистики обучения.
 
 
Компонент ```DecisionRequester``` автоматически запрашивает решения для экземпляра агента через регулярные промежутки времени (в шагах).

```Behavior Parameters``` - Компонент для настройки поведения экземпляра агента и свойств мозга, этот компонент необходим для того, чтобы "тренер" знал кого, тренировать, в полях этого компонента настраивается, как агент "видит" окружение, какой у агента интерфейс, чем агент "думает", компонент связывает сущности, необходимые для обучения агента.

## Задание 3
### Доработайте сцену и обучите ML-Agent таким образом, чтобы шар перемещался между двумя кубами разного цвета. Кубы должны, как и в первом задании, случайно изменять координаты на плоскости.

## Выводы
В выводах к работе дайте развернутый ответ, что такое игровой баланс и как системы машинного обучения могут быть использованы для того, чтобы его скорректировать.

Игровой балнс - это совокупность некоторых взаимовлияющих параметров игры и её элементов, которая создаёт запланированный разработчиком игровой опыт. Чтобы скорректировать игровой баланс могут быть использованы системы машинного обучения, они могут регулировать поток даннных в экономическом контексте (включать/выключать "Краны", "Дыры", менять цены, реагировать на уровни инфляции), регулировать "нагрузку" на игрока (запускать различные события (показатель события на еденицу времени, чтобы игрок не скучал и не уставал играть; влиять на шанс получить более ценный лут, снаряжение; давать игроку трудный, но посильный вызов (состав и многочисленность противников)). Главное преймущество использования средств машинного обучения  в том, что игра "подстраивается" под конкретного игрока, чтобы он получал именно запланированный разработчиком игровой опыт.
## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
