# Руководство по миграции

Версия 1.0 слегка отличается от 0.5.2 и более ранних версий. Эти изменения затрагивают перемещение, и, в большей степени, отрисовку. Здесь собраны некоторые советы по обновлению проектов с 0.x на 1.x.

## Основыне изменения

### Перемещение

Перемещение в основном такое же, но нужно отметить, что некоторые переменные переименованы, и их старые варианты устаренвшие и работают чуть медленнее:

* `dir` теперь `direction`;
* `spd` → `speed`;
* `grav` → `gravity`;
* `dravdir` → `gravityDir`.
Переменные вроде `speed`, `gravity` всеё ещё отражают число пикселей, добавляемых каждый кадр, однако, т.к. частота кадров по умолчанию теперь 60, потому нужно будет резать их на 2 для 30 FPS.

Дополнительно представлены 2 новых переменных: `hspeed` и `vspeed`, в которые можно писать и читать.

Под капотом новая система перемещения теперь работает на вертикальной и горизонатльной скоростях, а не на направлении и полной скорости.
Some inconsistencies may arise, though, especially with specific orders of execution:

```js
this.speed = 4;
this.direction = 90;
```

Everything is ok, `this.vspeed` is now -4.

```js
this.direction = 90;
this.speed = 4;
```

`this.direction = 90` makes no sense here, because `this.vspeed` and `this.hspeed` are equal to 0 and this rotation had no effect.

When making incremental movement without default variables, or when adding acceleration, you should multiply your numbers with `ct.delta`. So, instead of this:

```js
this.speed += 0.5;
this.x -= 10;
```

You should write:

```js
this.speed += 0.5 * ct.delta;
this.x -= 10 * ct.delta;
```

It is not necessary, yet recommended, as it helps to provide consistent movement with any framerate. 

`this.move();` utilizes `ct.delta`, so the default movement system will be consistent on every framerate by default.

### Transformations (`Cannot create property '_parentID' on boolean 'true'`)

Writing `this.transform = true;` will break your game now, as `transform` is now an object.

Instead of writing this:

```js
this.transform = true;
this.tx = 0.5;
this.ty = -1;
this.tr = 45;
this.ta = 0.5;
```

You should write this:

```js
this.scale.x = 0.5;
this.scale.y = 0.5;
this.rotation = 45;
this.alpha = 0.5;
```

### View boundaries

Instead of `ct.room.width` and `ct.room.height` use `ct.viewWidth` and `ct.viewHeight` only. These are different concepts now, and `ct.room.width` and `ct.room.height` changes over time.

::: warning
Stuff changed in v1.3. [See the migration guide](/migration-1.2to1.3.html). Shortly, `ct.room.width` is `ct.camera.width` now, and `ct.room.height` is `ct.camera.height`, but there are nuances once you start scaling or rotating the camera.
:::

### Timers
Instead of:

```js
this.shootTimer--;
```

Better write:

```js
this.shootTimer -= ct.delta;
```

## Drawing

First off: you can't directly draw in the Draw event now. Instead, you should create an object to draw (e.g. in the On Create event), and add it to your room or attach to an object, forming some kind of a widget.

### Drawing text labels

Instead of:

```js
ct.styles.set('ScoreText');
ct.draw.text('Score: ' + this.score, 20, 20);
ct.styles.reset();
```

You should write this to your On Create code:

```js
this.scoreLabel = new PIXI.Text('Score: ' + this.score, ct.styles.get('ScoreText'));
this.scoreLabel.x = this.scoreLabel.y = 20;
this.addChild(this.scoreLabel);
```

And update the label in the Draw event:

```js
this.scoreLabel.text = 'Score: ' + this.score;
```

### Drawing geometry

For this, use [PIXI.Graphics](https://pixijs.download/release/docs/PIXI.Graphics.html). Its API is similar to HTMLCanvas API, and one PIXI.Graphics object can contain more than one shape.

Example (On Create event):

```js
var overlay = new PIXI.Graphics();
overlay.beginFill(0x5FCDE4);
overlay.drawRect(0, 0, 59, 48);
overlay.endFill();
overlay.alpha = 0.65;

this.addChild(overlay);
```

### Drawing Healthbars, Mana Bars, etc.

For these, consider using built-in [9-slice scaling](https://en.wikipedia.org/wiki/9-slice_scaling). You should use an image that can be stretched horizontally and/or vertically, like this:

![](./../images/migrationBarSource.png)

Add this to your On Create code:

```js
this.healthBar = new PIXI.mesh.NineSlicePlane(
    ct.res.getTexture('Healthbar', 0),
    8, 8, 8, 16); /* this can be also written in one line */
this.addChild(this.healthBar);
this.healthBar.x = this.healthBar.y = 32; /* where to place this bar */
this.healthBar.height = 64;
this.healthBar.width = ct.game.health * 2; // Assuming that the max health is 100 and you want 100×2 = 200px wide bar
```

And update it each step with this code:	

```js
this.healthBar.width = ct.game.health * 2;
```

![](./../images/migrationBars.gif)

Constants `8, 8, 8, 16` tell which areas should not be stretched, in this order: on the left side, on the top, on the right and on the bottom.

Backgrounds for these bars can be made in the same way.

### Drawing Static Images

Two ways to do that exist:

1. Creating a new template that will display the needed image;
2. Or creating a PIXI.Sprite and adding it to the room (or to the object).

The second approach can be made in this way:

```js
this.coinIcon = new PIXI.Sprite(ct.res.getTexture('coinGold', 0));
this.coinIcon.x = 20;
this.coinIcon.y = this.y + 35;
this.addChild(this.coinIcon);
```

## Changes from 1.0.0-next-1 to 1.0.0-next-2

For working with different resolutions, you should now use `ct.viewWidth` and `ct.viewHeight` instead of `ct.width` and `ct.height`. The latter now mean the size of a drawing canvas, which doesn't always match with your viewport, especially when using new sizing modes of `ct.fittoscreen`.

## Changes in 1.0.0-next-3

### Keyboard and mouse support

ct.js now uses Actions for mapping a user input with game events. You can read about Actions [here](/actions.html). Because of that, `ct.mouse` dropped out of the core and is now a catmod.

All old projects will automatically work with `ct.mouse.legacy` and, if needed, `ct.keyboard.legacy`. They reflect the previous behaviour of these modules and should not cause compatibility issues.

The new `ct.keyboard` doesn't have `ct.keyboard.pressed`, `ct.keyboard.down`, and `ct.keyboard.released`. Instead, it fully relies on the new Actions system.

### Graphics -> Textures

Graphics, graphic assets, etc. are now called "Textures".

Now, instead of writing `this.graph = 'Sosiska';` in your code to change a texture, you should write `this.tex = 'Sosiska;`.
