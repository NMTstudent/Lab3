# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил:
- Батраков Дмитрий Антонович
- НМТ212701
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | # | 20 |
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

## Задание 3
### Доработайте сцену и обучите ML-Agent таким образом, чтобы шар перемещался между двумя кубами разного цвета. Кубы должны, как и в первом задании, случайно изменять координаты на плоскости.

## Выводы
Абзац умных слов о том, что было сделано и что было узнано.

В выводах к работе дайте развернутый ответ, что такое игровой баланс и как системы машинного обучения могут быть использованы для того, чтобы его скорректировать.

Игровой балнс - это совокупность некоторых взаимовлияющих параметров игры и её элементов, которая создаёт запланированный разработчиком игровой опыт. Чтобы скорректировать игровой баланс могут быть использованы системы машинного обучения, они могут регулировать поток даннных в экономическом контексте (включать/выключать "Краны", "Дыры", реагировать на уровни инфляции), регулировать "нагрузку" на игрока (запускать различные события (показатель события на еденицу времени, чтобы игрок не скучал и не уставал играть; влиять на шанс получить более ценный лут, снаряжение; давать игроку трудный, но посильный вызов).

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
