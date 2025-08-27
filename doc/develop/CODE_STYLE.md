# 项目代码风格规范

Kubejs:来自整合包[Create-Astral](https://github.com/Laskyyy/Create-Astral/blob/Astral-Experimental/CODE_STYLE.md)，为1.20版本，我们包为1.21.1kjs7会有所修改

# 代码风格

为了让源码更易读，请尽量遵循以下指南：

# 目录[变量命名](#1-变量命名)

1. [脚本编写](#2-脚本编写)
2. [类型检查](#3-类型检查)

   1. [设置](#31-设置)
   2. [类型使用](#32-类型使用)
   3. [推荐 VSCode 设置](#33-推荐-vscode-设置)

## 1. 变量命名

- 变量名应能描述变量的用途。

```js

// 推荐

items.forEach((item) => {

  //...

});


// 不推荐

items.forEach((e) => {

  //...

});

```

- 变量应使用 `let` 声明，采用 camelCase 命名：

```js

let sandStack = Item.of("minecraft:sand", 64);

```

- 常量应使用 `const` 声明，命名可用 camelCase 或 SCREAMING_SNAKE_CASE：

```js

const incompleteTransitionalCircuit = "createastral:incomplete_transitional_electronic_circuit";

const BUCKET = 81000;

```

- 函数应使用 camelCase 命名：

```js

function itemCount(itemName, count) {

  // ...

}

```

- 类应使用 PascalCase 命名。

  KubeJS 不支持 `class` 关键字，因此需使用构造函数代替。

```js

function SequencedAssemblyBuilder(event, io) {

  this._event = event;

  // ...

}

```

- Java 类名前应加 `$` 符号。

```js

const $DeferredRegister = java("dev.architectury.registry.registries.DeferredRegister");

```

## 2. 脚本编写

- 每个脚本都应包裹在一个立即执行函数表达式（IIFE）中。这样可以防止变量泄漏到全局作用域，除非你有意通过 `global` 对象赋值：

```js

// IIFE 示例

(function () {

  const private = "我不是全局变量，只能在本脚本访问！";

  global.globalString = "我是全局变量，所有脚本都能访问！";

})();

// IIFE 也可以命名

(function myScript() {

  // ...

})();

```

- 对于长的配方列表，使用对象数组，不要用数组嵌套数组（元组），这样更易用 JSDoc 注释。

```js

// 推荐

const hauntingRecipes = [

  {

    input: "naturalist:snail_shell",

    output: "minecraft:nautilus_shell",

  },

  // ...

];


hauntingRecipes.forEach((recipe) => {

  // ...

});

```

```js

// 不推荐

const hauntingRecipes = [

  ["naturalist:snail_shell", "minecraft:nautilus_shell"],

  // ...

];


hauntingRecipes.forEach((recipe) => {

  // ...

});

```

- 多个物品堆叠时，优先用 `Item.of(item, count)`，不要用 `${count}x ${item}`，尤其是动态创建堆叠时。这样更易用 ProbeJS 类型检查。

```js

// 推荐

{

  input: Item.of("techreborn:coal_dust", 2),

  output: "techreborn:coal_dust",

},

// 不推荐

{

  input: "2x minecraft:charcoal",

  output: "minecraft:coal",

},

```

- 尽量使用模板字符串，不要用字符串拼接。

  这样可以让 TypeScript 检查所有模板字符串的类型。

```js

// TypeScript 能用 JSDoc 注释检查 JavaScript 代码类型。

// 这里声明数组不会改变，因此每个结果都能类型检查。

const gems = /** @type {const} */ (["peridot", "red_garnet", "ruby", "sapphire", "yellow_garnet"]);

// 推荐

gems.forEach((gem) => {

  HIDDEN_ITEMS.push(`techreborn:${gem}_storage_block`);

});


// 不推荐

gems.forEach((gem) => {

  HIDDEN_ITEMS.push("techreborn:" + gem + "_storage_block");

});

```

- 本仓库在 `global` 对象中定义常量和通用代码。

  - `global.server` 存储服务端通用常量和函数。
  - `global.fluids` 存储与流体数量相关的常量，如 `BUCKET` 或 `mB`。
  - `global.startup` 存储启动相关的常量和函数。
- 访问常量时，在 IIFE 内部解构对象：

```js

(function myScript() {

  const { createSequencedAssembly } = global.server

  const { BUCKET, mB } = global.fluids


  // ...

})();

```

## 3. 类型检查

### 3.1. 设置

你可以选择让脚本通过 ProbeJS 进行类型检查，方法如下：

- 在整合包中添加 ProbeJS：https://www.curseforge.com/minecraft/mc-mods/probejs/files/4365861
- 启动 Minecraft
- 为获得最佳类型导出，依次执行：

  - `/kubejs reload client_scripts`
  - `/kubejs reload server_scripts`
  - `/kubejs reload startup_scripts`
  - 尝试在床上睡觉
  - 用物品右键点击方块
  - 用物品左键点击方块
- 执行 `/probejs dump`
- 类型声明应出现在：

```

<instance>/minecraft/kubejs/probe/generated

```

由于 ProbeJS 会为包含短横线的方法（如 Mixin 生成）导出类型，导致生成的 `globals.d.ts` 有语法错误（JavaScript 方法名不能有短横线）。

- 用支持正则替换的文本编辑器（如 VS Code）打开 `globals.d.ts`。
- 点击“使用正则表达式”按钮。
- 用正则表达式将所有 `(?<!")((?=\S*['-])(?:[a-zA-Z$\d_'-]+))(?!")(?=\()` 替换为 `"$1"`。
- 应无语法错误。

另外，ProbeJS 会错误地尝试为 `global` 声明类型：

- 打开 `constants.d.ts`。
- 删除或注释掉 `type global = ...`。
- 现在 `global` 应能正确推断类型。

然后：

- 将 Minecraft 实例中的 `probe` 文件夹复制到仓库中。`probe` 已在 `.gitignore`，不会被提交。
- 用以下命令阻止 `kubejs/jsconfig.json` 的更改被提交：

```sh

git update-index --assume-unchanged kubejs/jsconfig.json

```

（如需恢复提交，执行：）

```sh

git update-index --no-assume-unchanged kubejs/jsconfig.json

```

- 在 `kubejs/jsconfig.json` 中，将 `checkJs` 字段从 `false` 改为 `true`。

这样，每个 JavaScript 文件都会用 ProbeJS 和本仓库自定义声明进行类型检查！

```js

(function myCoolScript() {

  let a = 5;

  a = "string";

  // ^ 错误：类型 'string' 不能赋值给类型 'number'。

});

```

### 3.2. 类型使用

TypeScript 能从用法推断基础类型，但很多情况下需用 JSDoc 注释显式声明变量类型。

有用链接：

- [JSDoc](https://jsdoc.app/)
- [TypeScript 对 JSDoc 的扩展](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html)

常用语法示例：

#### 设置变量类型

```js

/** @type {Internal.IngredientJS_[]} */

let HIDDEN_ITEMS = [

  "ae2:silicon",

  "extended_drawers:t1_upgrade",

  "extended_drawers:t2_upgrade",

  //...

]

// 现在 HIDDEN_ITEMS 是 Internal.IngredientJS_ 类型的数组。

```

#### 定义类型

##### 类型

```ts

/** @typedef {"create:blaze_burner" | "createaddition:liquid_blaze_burner"} Burner */


// 等价于 TypeScript 语法：

type Burner = "create:blaze_burner" | "createaddition:liquid_blaze_burner"


// 现在可以在脚本中使用该类型：

/** @type {Burner} */

let burner = "create:blaze_burner"

// 只能赋值为 "Burner" 类型定义的字符串

```

##### 接口

接口声明对象结构。可用作变量或常量类型。

```ts

/**

 * @typedef CrushingRecipeToBecomeGrinding

 * @property {object} input

 * @property {Special.Item} input.item

 * @property {object} output

 * @property {Special.Item} output.item

 * @property {number} output.count

 * @property {number} [time]

 * @property {number} [power]

 */

// 方括号内属性为可选。


// 等价于 TypeScript 语法：

interface CrushingRecipeToBecomeGrinding {

  input: { item: Special.Item };

  output: { item: Special.Item; count: number };

  time?: number;

  power?: number;

}


// 现在可以在脚本中使用该类型：


/** @type {CrushingRecipeToBecomeGrinding[]} */

const crushingRecipesToBecomeGrinding = [

  //...

]

```

#### 声明列表为不可变

```js

// @satisfies {string[]} 确保数组为字符串数组但不改变类型。

// @type {const} 声明数组不会改变。

// 这样 TypeScript 能更有效地检查模板字符串的有效性。

// 注意圆括号！

/** @satisfies {string[]} */

const names = /** @type {const} */ ([

  "helmet",

  "chestplate",

  "leggings",

  "boots",

  "sword",

  "pickaxe",

  "shovel",

  "axe",

  "hoe",

]);

```

#### 常用类型

- KubeJS 函数用到的物品类型：

  - `Special.Item` - 只接受物品字符串，
  - `Internal.ItemStackJS_` - 接受 `Item` 和其他物品字符串。
  - `Internal.IngredientJS_` - 包含大多数可作为材料的类型。

优先用 `Item.of("minecraft:apple", 5)`，不要用 `5x minecraft:apple`，后者类型检查会报错。

- KubeJS 函数用到的流体类型：

  - `Special.Fluid` - 只接受流体字符串，
  - `{fluid: Special.Fluid, amount: number}` - 接受符合该结构的对象。
  - `Internal.IngredientJS_` - 包含大多数可作为材料的类型。

### 3.3. 推荐 VSCode 设置

- **TypeScript › Tsserver › Experimental: Enable Project Diagnostics**：启用。

启用后，整个项目都会类型检查，而不仅是打开的文件。

- **JavaScript › Inlay Hints › Function Like Return Types**：启用。
- **JavaScript › Inlay Hints › Parameter Types**：启用。
- **JavaScript › Inlay Hints › Property Declaration Types**：启用。
- **JavaScript › Inlay Hints › Variable Types**：启用。

这些选项会显示基于推断类型
