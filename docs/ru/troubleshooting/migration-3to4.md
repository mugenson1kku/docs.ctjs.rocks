# Руководство по миграции для ct.js 4.0.

## Ct.js потерял `ct.`! 🙀

Все команды вида `ct.something` теперь `something`, с парой исключений:

* `ct.sound` теперь `sounds`, во множественном числе.
    * `ct.sound.spawn` теперь `sounds.play`
    *  Некоторые опции и методы могут называться иначе; подробнеее смотрите [здесь](../sounds.md).
* `ct.delta` теперь `u.delta`, `ct.deltaUi` - `u.deltaUi`.
* `ct.room` теперь `rooms.current`.
* `ct.pixiApp` отныне просто `pixiApp`.
* `ct.roomWidth` т `ct.roomHeight` теперь `rooms.current.viewWidth` и `rooms.current.viewHeight`.
* `ct.speed` теперь `settings.targetFps`.

**Пример.** Старый код:

```js
// A snippet from the catsteroids demo
this.targetx = ct.random.range(75, ct.camera.width - 75);
this.targety = ct.random.range(75, 300);
ct.tween.add({
    obj: this,
    fields: {
        x: this.targetx,
        y: this.targety
    },
    duration: 1500
});
```

Новый код:

```js
// A snippet from the catsteroids demo
this.targetx = random.range(75, camera.width - 75);
this.targety = random.range(75, 300);
tween.add({
    obj: this,
    fields: {
        x: this.targetx,
        y: this.targety
    },
    duration: 1500
});
```

### `camera` теперь неизменяемая

Теперь нельзя присвойить новую камеру через переменную `camera`.

## Новые значения для отслеживания, изменения в `this.move()` и фонах

Помимо `u.delta` и `u.deltaUi`, в ct.js теперь ещё есть `u.time` и `u.timeUi`, которые исчистяются в секундаж и показывают пройденное время между 1-м, последним и текущим кадром. Рекомендуется применять эти значения для скорости или других зависимых от времени величин, вместо `u.delta` и `u.deltaUi, ведь последние две не дают плавного передвижения при смене ограничения частоты кадров в игре. 

* Из-за этого метод `this.move()` теперь использует новые значения, и нужно умножать значение скорости на максимальный FPS, объявленный в настройках проекта (по умолчанию - 60).

* Это также за трагивает `place.moveSmart` и `place.moveBullet`, а также скорость движения фонов.

* With acceleration values like those in `this.gravity` and `this.addSpeed`, you will need to multiply these with your max framerate **twice** (which is 3600 with default framerate), because physics.

* `u.delta` и `u.deltaUi` теперь устаревшие, но остаются доступными.

## Котомод FitToScreen теперь часть основной библиотеки ct.js

Viewport settings are moved to project's render settings. Moreover, you can change viewport settings on the fly in the game with the new [settings API](/settings.md#settings-viewmode)!

## Скелетная анимация более не поддерживается

В ct.js обновлена лежащая в основе библиотека Pixi.jss, and though it brought various benefits for development and rendering performance, the DragonBones runtime is no longer supported by it, thus the skeletal animations are no longer supported in ct.js. Возможно, в будущем появится поддержка другого рантайма, если его лицензия будет совместимя с Лицензией MIT.

## Pixi.js обновлён с 5.3 до 7.1

Это несёт в себе некоторые критичные изменения и устаревания из графической библиотеке в основе ct.js. Однако, это никак не повлияет на проект, в котором API Pixi.js используется напрямую.

## Котомоды

Опция `useUiDelta` в `tween.add` переименована в `isUi` для соблюдения названий как в других частях ct.js.

Инъекция `/*!%start%*/` была удалена. Теперь нужно писать код в файл `index.js`.

Котомоды `mouse` и `touch` были удалены. Вместо них используйте модуль `pointer`, или встроенные события, которые можно добавить в список событий шаблонов.

## ct.u.ext (aka ct.u.extend) were removed

Use `Object.assign(target, valuesObject)` instead — it works the same.
