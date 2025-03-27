# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ В РАЗРАБОТКЕ ИГР

Отчет по лабораторной работе #3 выполнил:

- Торопов Матвей Николаевич
- РИ230931

Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

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

Установить необходимое программное обеспечение, которое пригодится для создания интеллектуальных моделей на Python. Рассмотреть процесс установки игрового движка Unity для разработки игр.

## Задание 1

### Найти переменные, влияющие на геймплей в Dragon Picker и предложить варианты их изменения

На движение дракона влияют 2 переменные:

- Направление движения
- Скорость движения

Помимо этого, есть ещё 2 переменные в игре:

- Период выпадения яйца
- Скорость падения яйца

Направление движения не влияет на сложность (какая нам разница, в какую сторону движется дракон)  
А вот от скорости движения дракона и периода выпадения яйца напрямую зависит место появления яйца

Собственно, на сложность игры напрямую влияют следующие переменные:

- Скорость передвижения дракона
- Период выпадения яйца
- Скорость падения яйца

Для разделения на уровни можно ввести величину Необходимое количество яиц», название которой говорит само за себя: это необходимое количество яиц, которое нужно собрать для перехода на следующий уровень

Для моделирования зависимости переменных от уровня сложности я написал следующую программу:

```py
import numpy as np
import matplotlib.pyplot as plt

levels = np.arange(1, 11)

egg_speed = np.linspace(0, 4, 10)**2
egg_period = np.linspace(2.0, 1.2, 10)**2
dragon_speed = np.linspace(1, 4, 10)**2
egg_count = np.linspace(3, 8, 10)**2

fig, ax = plt.subplots(4, 1, figsize=(8, 12))

ax[0].plot(levels, egg_speed, marker='o', linestyle='-', color='red', label='Скорость яйца')
ax[0].set_title("Скорость яйца от уровня")
ax[0].set_xlabel("Уровень сложности")
ax[0].set_ylabel("(м/с)")
ax[0].grid(True)
ax[0].legend()

ax[1].plot(levels, egg_period, marker='o', linestyle='-', color='blue', label='Период падения')
ax[1].set_title("Период падения яйца от уровня")
ax[1].set_xlabel("Уровень сложности")
ax[1].set_ylabel("(сек.)")
ax[1].grid(True)
ax[1].legend()

ax[2].plot(levels, dragon_speed, marker='o', linestyle='-', color='green', label='Скорость дракона')
ax[2].set_title("Скорость дракона от уровня")
ax[2].set_xlabel("Уровень сложности")
ax[2].set_ylabel("(м/с)")
ax[2].grid(True)
ax[2].legend()

ax[3].plot(levels, egg_count, marker='o', linestyle='-', color='purple', label='Количество яиц')
ax[3].set_title("Количество яиц")
ax[3].set_xlabel("Уровень сложности")
ax[3].set_ylabel("(шт.)")
ax[3].grid(True)
ax[3].legend()

plt.tight_layout()
plt.show()

import gspread
import numpy as np
gc = gspread.service_account(filename='da-in-gamedev-b2ccdbaa1a56.json')
sh = gc.open("da-in-gamedev-lab3")

def update_sheet(i, es, ep, ds, ec):
    sh.sheet1.update(('A' + str(i)), str(round(es, 3)))
    sh.sheet1.update(('B' + str(i)), str(round(ep, 3)))
    sh.sheet1.update(('C' + str(i)), str(round(ds, 3)))
    sh.sheet1.update(('D' + str(i)), str(round(ec, 3)))

for i in range(10):
    update_sheet(i+1, egg_speed[i], egg_period[i], dragon_speed[i], egg_count[i])

```

Соответственно, как результат мы получаем четыре графика matplotlib и данные в Google SpreadSheets

![python-matplotlib-part1](python-matplotlib-part1.png)
![python-matplotlib-part2](python-matplotlib-part2.png)
![google-spreadsheets](google-spreadsheets.png)

## Задание 2

### Создайте 10 сцен на Unity с изменяющимся уровнем сложности

Имея в Google Spreadsheets данные о значениии переменных, мы можем в Unity написать скрипт, который будет «вытягивать» эти данные, и соответственно менять уровни сложности, в зависимости от текущего уровня.

Вот скрипты, которые я использовал для получения и хранения данных об уровне сложности

```cs
public class LevelData
{
	public LevelData(float EggSpeed, float EggPeriod, float DragonSpeed, float EggCountNeeded)
	{
		this.DragonSpeed = DragonSpeed;
		this.EggSpeed = EggSpeed;
		this.EggPeriod = EggPeriod;
		this.EggCountNeeded = EggCountNeeded;
	}

	public float DragonSpeed { get; }
	public float EggSpeed { get; }
	public float EggPeriod { get; }
	public float EggCountNeeded { get; }

	public override string ToString()
	{
		return $"ds {DragonSpeed}; es {EggSpeed}; ep {EggPeriod}; ec {EggCountNeeded}";
	}
}
```
```cs
using System;
using System.Collections;
using System.Collections.Generic;
using SimpleJSON;
using UnityEngine;
using UnityEngine.Networking;
using UnityEngine.SceneManagement;

public class LevelInfo : MonoBehaviour
{
	public static LevelInfo instance;
	public LevelData CurrentLevelData;
	public bool GameReady = false;
	private List<LevelData> _levels = new();

	public void Awake()
	{
		instance = this;
		DontDestroyOnLoad(this);
		StartCoroutine(GoogleSpreadSheet());
		SceneManager.activeSceneChanged += OnSceneChange;
	}

	private void OnSceneChange(Scene current, Scene next)
	{
		if (next.buildIndex == 0) return;
		CurrentLevelData = _levels[next.buildIndex - 1];
		Debug.Log(CurrentLevelData);
	}

	IEnumerator GoogleSpreadSheet()
	{
		var uwr = UnityWebRequest.Get(
			"https://sheets.googleapis.com/v4/spreadsheets/1zhoqeoifxE0ZUfAjCogdQtAoijlDh_FqckWbScd3jm4/values/Лист1?key=AIzaSyA3SxQjXEfSFlvXw8wTvkl6SpQdWiYbzJU");
		yield return uwr.SendWebRequest();
		var rawJson = JSON.Parse(uwr.downloadHandler.text);
		foreach (var row in rawJson["values"])
		{
			var rowData = JSON.Parse(row.ToString())[0].AsStringList;
			Debug.Log(rowData[0] + " " + rowData[1] + " " + rowData[2] + " " + rowData[3]);
			var newLevelData = new LevelData(
				float.Parse(rowData[0].Replace('.', ',')),
				float.Parse(rowData[1].Replace('.', ',')),
				float.Parse(rowData[2].Replace('.', ',')),
				float.Parse(rowData[3].Replace('.', ',')));
			_levels.Add(newLevelData);
		}

		CurrentLevelData = _levels[0];
		GameReady = true;
	}
}
```

Отредактировав некоторые предоставленные скрипты и добавив новые сцены в билд, мы можем протестировать наши варианты изменения сложности.

![unity-scenes-build](unity-scenes-build.png)

## Задание 3

### Выполнить работу используя знания о связи Python — Google SpreadSheets — Unity

Работа выполнена

## Вывод

Я научился определять важные для баланса игры переменные и подбирать оптимальные пути изменения к ним. Для этого я воспользовался средствами Google SpreadSheets, Python (matplotlib и numpy), а так же Unity

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
