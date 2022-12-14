# DA-In-GameDev-lab3
# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил(а):
- Намесников Глеб Артёмович
- РИ210912
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
Ознакомиться с основными операторами зыка Python на примере реализации линейной регрессии.

## Задание 1
### Реализовать систему машинного обучения в связке Python - Google-Sheets – Unity.
Ход работы:

1) В созданный проект были добавлены необходимые пакеты ML Agents, которые далее появлись в Components ![image](https://user-images.githubusercontent.com/103383207/198278175-5b3a639d-39c6-439c-9fe7-8252b077dfe9.png)
2) В ходе работы с Anaconda Prompt был активирован ML Agent, а так же установлены необходимые библиотеки ML Agents и Torch ![image](https://user-images.githubusercontent.com/103383207/198279091-8926e132-88f7-4547-858c-e23d8a1291f7.png)
![image](https://user-images.githubusercontent.com/103383207/198279124-50a0bb47-e0d8-40e1-8b7a-215991088764.png)
![image](https://user-images.githubusercontent.com/103383207/198279170-06ba8582-4e23-4b35-9eef-c33feadec69d.png)
3) Созданы необходимые обьекты на сцене, а к сфере Roller Agent применен скрипт, а так же компоненты Rigid Body, Decision Requester, Behavior Parameters. ![image](https://user-images.githubusercontent.com/103383207/198279504-71ad97b6-cb86-478c-bd87-11ee961672bc.png)

```cs

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

5) Запущен ML Agent через Anaconda Prompt, начата тренировка. ![image](https://user-images.githubusercontent.com/103383207/198279914-43a3bca6-0184-47d5-9ac8-1a1fc99b2c7f.png)
![image](https://user-images.githubusercontent.com/103383207/198279975-c22ea79a-611f-49b6-be83-d3f7b33da54c.png)

6) Далее были установлены 3 поля, 9 полей и наконец 27 полей. Было замечено, что чем больше одинаковых полей, тем быстрее происходит обучение из-за увеличения итераций. ![2022-10-27-18-32-21](https://user-images.githubusercontent.com/103383207/198299366-a4c57de1-db9d-4ccd-abb4-011dc642a9f8.gif)


С нарастанием шагов, сфера начала двигаться очень плавно и более грамотно и ловко догонять таргетный куб.

## Задание 2

### Описание конфигурационного файла.

```cs

 behaviors: 
  RollerBall: #Behaviour Name в компонентах
    trainer_type: ppo #тип тренировки Proximal Policy Optimization
    hyperparameters: #обучение с задаваемыми гипер параметрами
      batch_size: 10 #кол-во опытов
      buffer_size: 100 #кол-во опыта, необходимое для обновления модели
      learning_rate: 3.0e-4 #скорость обучения
      beta: 5.0e-4 #параметр энтропии, позволяющий корректно использовать пространство действий
      epsilon: 0.2 #коэфициент развития политики обучения
      lambd: 0.99 #параметр лямбда, который расчитывает обобщенную оценку преимущества
      num_epoch: 3 #количество проходов через буфер опыта во время градиентного спуска
      learning_rate_schedule: linear #параметр, определяющий скорость обучения с течением определенного времени
    network_settings:
      normalize: false #применяется ли нормализация к данным векторного наблюдения или же нет
      hidden_units: 128 #количество единиц в подключенных слоях нейронной сети.
      num_layers: 2 #кол-во слоев
    reward_signals: #настройки наград
      extrinsic: #внешние награды
        gamma: 0.99 #коэф. будущих вознаграждений
        strength: 1.0 #коэф, на который можно умножить вознаграждение, получаемое от окружающей среды
    max_steps: 500000 #максимальное кол-во итераций, которых может выполнить агент
    time_horizon: 64 #кол-во шагов сбора опыта для каждого агента до добавления его в буфер опыта
    summary_freq: 10000 #кол-во опыта, соответствующее частоте обновления агента

```

## Задание 3



## Выводы

Игровой баланс — в играх (спортивных, настольных, компьютерных и прочих) субъективное «равновесие» между персонажами, командами, тактиками игры и другими игровыми объектами. Игровой баланс — одно из требований к «честности» правил. Мы используем машинное обучение, чтобы сделать логику и передвижение различных обьектов (NPC, врагов и т.д) более живыми и естественными. В ходе данной лабораторной работы мы как раз увидели, что шар, пройдя достаточное количество итераций начинает двигаться более плавно и естественно.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
