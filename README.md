# StandardConf

## 配置环境

- Init
- Git Commit Message
- TypeScript
- ESLint
- Prettier
- Lint Staged

### Init

采用 NPM 可以对任何普通的项目进行初始化操作，执行 [`npm init`](https://docs.npmjs.com/cli/init) 会在项目根目录下生成 `package.json` 包描述文件。

> **温馨提示**：更多详细项目变更可以查看 Commit。

### Git Commit Message

[Commitizen](https://github.com/commitizen/cz-cli) 是一个规范 Git 提交说明（Commit Message）的 CLI 工具，具体如何配置可查看 [Cz 工具集使用介绍](https://juejin.im/post/5cc4694a6fb9a03238106eb9)。本项目中主要使用了以下功能：

- [cz-customizable](https://github.com/leonardoanalista/cz-customizable)
- [commitlint](https://commitlint.js.org/#/)
- [conventional-changelog](https://github.com/conventional-changelog/conventional-changelog/tree/master/packages/conventional-changelog)

配置以后会产生以下一些特性：

- 使用 `git cz` 代替 `git commit` 进行符合 Angular 规范的 Commit Message 信息提交
- 代码提交之前会通过 [husky](https://github.com/typicode/husky) 配合 git hook 进行提交信息校验，一旦提交信息不符合 Angular 规范，则提交会失败
- 执行 `npm run log` 会在根目录下自动生成 `CHANGELOG.md` 版本日志

> **温馨提示**：如果不知道什么是 CLI （命令行接口），可查看 [使用 NPM 发布和使用 CLI 工具](https://juejin.im/post/5eb89053e51d454de54db501)。

### TypeScript

本项目会构建输出 CommonJS 工具包（npm 包）供外部使用，采用 TypeScript 设计并输出声明文件有助于外部更好的使用该资源包。TypeScript 编译采用官方文档推荐的 Gulp 工具，配合 [gulp-typescript](https://github.com/ivogabe/gulp-typescript) 和 [tsconfig.json](https://www.tslang.cn/docs/handbook/tsconfig-json.html) 配置文件，可快速进行项目构建。在根目录下新建 `tsconfig.json` 文件并新增以下配置：

```javascript
{
  "compilerOptions": {
    // 指定 ECMAScript 目标版本 "ES3"（默认）， "ES5"， "ES6" / "ES2015"， "ES2016"， "ES2017" 或 "ESNext"。
    "target": "ES5",
    // 构建的目标代码删除所有注释，除了以 /!* 开头的版权信息
    "removeComments": true,
    // 可配合 gulp-typescript 生成相应的 .d.ts 文件
    "declaration": true,
    // 启用所有严格类型检查选项。启用 --strict 相当于启用 --noImplicitAny, --noImplicitThis, --alwaysStrict， --strictNullChecks, --strictFunctionTypes 和 --strictPropertyInitialization
    "strict": true,
    // 禁止对同一个文件的不一致的引用
    "forceConsistentCasingInFileNames": true,
    // 报错时不生成输出文件
    "noEmitOnError": true
  }
}
```

> **温馨提示**：这里没有新增 `module` 配置信息，因为默认输出 CommonJS 规范，更多关于 TypeScript 配置信息可查看[TypeScript 官方文档 / 编译选项](https://www.tslang.cn/docs/handbook/compiler-options.html)。如果对于 CommonJS 和 ES6 规范的区别不是很清晰，这里有一篇非常好的文档可以供大家阅读：[ES modules: A cartoon deep-dive](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)。
同时在根目录下新建 `gulpfile.js` 文件：

```javascript
const gulp = require("gulp");
const ts = require("gulp-typescript");
const tsProject = ts.createProject("tsconfig.json");
// 输出 CommonJS 规范到 dist 目录下
gulp.task("default", function () {
  const tsResult = tsProject.src().pipe(tsProject());
  return tsResult.js.pipe(gulp.dest("dist"));
});
```

在 `package.json` 中新增 script 脚本：

```javascript
"scripts": {
  "build": "rimraf dist && gulp"
},
```

其中 [rimfaf](https://github.com/isaacs/rimraf) 用于构建之前清除 dist 目录文件内容。此时在 `src` 目录下新增 TypeScript 源码并使用 `npm run build` 命令可以进行项目构建并输出 CommonJS 规范的目标代码到 `dist` 目录下。

除此之外，此项目希望可以快速生成声明文件供外部进行代码提示，此时仍然可以借助 `gulp-typescript` 工具自动生成声明文件。在 `gulpfile.js` 中新增以下配置

```javascript
const gulp = require("gulp");
const ts = require("gulp-typescript");
const tsProject = ts.createProject("tsconfig.json");
const merge = require("merge2");
// 输出 CommonJS 规范到 dist 目录下
gulp.task("default", function () {
  const tsResult = tsProject.src().pipe(tsProject());
  return merge([
    tsResult.dts.pipe(gulp.dest("types")),
    tsResult.js.pipe(gulp.dest("dist")),
  ]);
});
```

修改 `build` 命令使其在构建之前同时可以删除 `types` 目录：

```javascript
"scripts": {
  "build": "rimraf dist types && gulp",
},
```

再次执行 `npm run build` 会在项目根目录下生成 `types` 文件夹，该文件夹主要存放自动生成的 TypeScript 声明文件。

需要注意发布 npm 包时默认会将当前项目的所有文件进行发布处理，但是这里希望发布的包只包含使用者需要的编译文件 `dist` 和 `types`，因此可以通过`package.json` 中的 [`files`](https://docs.npmjs.com/files/package.json#files)（用于指定发布的 NPM 包包含哪些文件） 字段信息进行控制：

```javascript
"files": [
  "dist",
  "types"
],
```

> **温馨提示**：发布的 npm 包中某些文件将忽视 `files` 字段信息，包括 `package.json`、`LICENSE`、`README.md` 等。
除此之外，如果希望发布的 npm 包通过 `require('algorithms-utils')` 或 `import` 形式引入时指向 `dist/index.js` 文件，需要配置 `package.json` 中的 [`main`](https://docs.npmjs.com/files/package.json#main) 字段信息：

```javascript
"main": "dist/index.js"
```

> **温馨提示**： 对于工具包使用全量引入的方式并不是一个好的选择，可以通过具体的工具方法进行按需引入。

### ESLint

#### ESLint背景

TypeScript 的代码检查工具主要有 TSLint 和 ESLint 两种。早期的 TypeScript 项目一般采用 TSLint 进行检查，TSLint 和 TypeScript 采用同样的 AST 格式进行编译，但主要问题是对于 JavaScript 生态的项目支持不够友好，因此 TypeScript 团队在 2019 年宣布全面转向 ESLint，更多关于转向 ESLint 的原因可查看：

- <https://medium.com/palantir/tslint-in-2019-1a144c2317a9>

- <https://github.com/microsoft/TypeScript/issues/30553>

TypeScript 和 ESLint 使用不同的 AST 进行解析，因此为了在 ESLint 中支持 TypeScript 代码检查需要制作额外的[自定义解析器](https://cn.eslint.org/docs/developer-guide/working-with-custom-parsers)（Custom Parsers，ESLint 的自定义解析器功能需要基于 [ESTree](https://github.com/estree/estree)），目的是为了能够解析 TypeScript 语法并转成与 ESLint 兼容的 AST。[@typescript-eslint/parser](https://github.com/typescript-eslint/typescript-eslint#getting-started--installation) 在这样的背景下诞生，它会处理所有 ESLint 特定的配置并调用 [@typescript-eslint/typescript-estree](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/typescript-estree) 生成 ESTree-compatible AST（需要注意的是不仅仅兼容 ESLint，也能够兼容 Prettier）。

`@typescript-eslint` 是一个 Monorepo 体系结构的仓库，采用 [Learn](https://github.com/lerna/lerna) 进行设计，除了上述提到的 NPM 包之外，还包含以下两个重要的 NPM 包：

- [@typescript-eslint/eslint-plugin](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/eslint-plugin): 配合 `@typescript-eslint/parser` 一起使用的 ESLint 插件，可以设置 TypeScript 的校验规则。

- [@typescript-eslint/eslint-plugin-tslint](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/eslint-plugin-tslint): TSLint 向 ESLint 迁移的插件。

> **温馨提示**：如果你正在使用 TSLint，并且你希望兼容 ESLint 或者向 ESLint 进行过渡（TSLint 和 ESLint 并存）， 可查看 [Migrating from TSLint to ESLint](https://github.com/typescript-eslint/typescript-eslint#migrating-from-tslint-to-eslint)。除此之外，以上所介绍的这些包发布时版本一致（为了联合使用的适配性），如果还有什么需要注意的话你可能需要关心一下 `@typescript-eslint` 对于 TypeScript 和 ESLint 的版本支持性，更多可查看该库包的 @typescript-eslint/parser 的仓库信息。

#### ESLint配置

从背景的介绍中可以理解，对于全新的 TypeScript 项目（直接抛弃 TSLint）需要包含解析 AST 的解析器 @typescript-eslint/parser 和使用校验规则的插件 @typescript-eslint/eslint-plugin，这里需要在项目中进行安装

```javascript
yarn add eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin -D
```

在根目录新建 `.eslintrc.js` 配置文件，并设置以下配置：

```javascript
module.exports = {
  root: true,
  parser: "@typescript-eslint/parser",
  plugins: ["@typescript-eslint"],
  extends: ["eslint:recommended", "plugin:@typescript-eslint/recommended"],
};
```

其中：

- `parser: '@typescript-eslint/parser'`：使用 ESLint 解析 TypeScript 语法

- `plugins: ['@typescript-eslint']`：在 ESLint 中加载插件 `@typescript-eslint/eslint-plugin`，该插件可用于配置 TypeScript 校验规则。

- `extends: [ ... ]`：在 ESLint 中使用[共享规则配置](https://cn.eslint.org/docs/developer-guide/shareable-configs)，其中 `eslint:recommended` 是 ESLint 内置的推荐校验规则配置（也被称作最佳规则实践），`plugin:@typescript-eslint/recommended` 是类似于 `eslint:recommended` 的 TypeScript 推荐校验规则配置。

> **温馨提示**：如果你稍微阅读一下 recommanded 源码你会发现，其实内部可以理解为推荐校验规则的集合。因此如果想基于 `@typescript-eslint/eslint-plugin` 进行自定义规则，则可参考 [TypeScript Supported Rules](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/eslint-plugin#supported-rules)。

配置完成后在 `package.json` 中设置校验命令

```javascript
"lint": "eslint src",
```

此时如果在 `src` 目录下书写错误的语法，执行 `npm run lint` 就会输出错误信息：

```javascript
> eslint src
C:\Code\Git\algorithms\src\greet.ts
  2:16  warning  Missing return type on function  @typescript-eslint/explicit-module-boundary-types
✖ 1 problem (0 errors, 1 warning)
```

> **温馨提示**：输出的错误信息是通过 [ESLint Formatters](https://cn.eslint.org/docs/user-guide/formatters/) 生成，查看 ESLint 源代码并调试可发现默认采用的是 [stylish](https://cn.eslint.org/docs/user-guide/formatters/#stylish) formatter。

#### ESLint 插件

如果不使用插件，很难发现写的代码可能存在 TypeScript 格式错误（除非手动 `npm run lint` 或监听代码的变更并实时运行 `npm run lint`），此时可以通过 VS Code 插件进行处理。安装 ESLint 插件后可进行代码的实时提示，具体如下图所示：

![ESLint Plugin.png](https://camo.githubusercontent.com/7f87d7fb16a0c01d7ff1a025e04e77460ee4363d/68747470733a2f2f70332d6a75656a696e2e62797465696d672e636f6d2f746f732d636e2d692d6b3375316662706663702f39626134323433666137633134303036393836363035393834623562376638317e74706c762d6b3375316662706663702d7a6f6f6d2d312e696d616765)

当然为了防止不需要被校验的文件出现校验信息，可以通过 `.eslintignore` 文件进行配置（例如以下都是一些不需要格式校验的配置文件）

```javascript
# gulp
gulpfile.js

# eslint
.eslintrc.js

# commitizen
commitlint.config.js

# jest
jest.config.js

# build
dist
```

此时可以发现之前执行 `lint` 命令的错误通过插件的形式可实时在 VS Code 编辑器中进行显示。除此之外，一些简单的 ESLint 格式错误（例如 多余的`;` 等）可通过配置 Save Auto Fix 进行保存自动格式化处理。具体 VS Code 的配置可参考 [ESLint 插件](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)的文档说明，这边应该需要进行如下配置：

``` javascript
"editor.codeActionsOnSave": {
  "source.fixAll": true,
  "source.fixAll.eslint": true
}
```

> **温馨提示**：VS Code 的配置分为两种类型（用户和工作区），针对上述通用的配置主要放在用户里，针对不同项目的不同配置则需要放入工作区进行处理。

#### ESLint 确保构建

VS Code 插件并不能确保代码上传或构建前无任何错误信息，此时仍然需要额外的流程能够避免错误。在构建前进行 ESLint 校验能够确保构建时无任何错误信息，一旦 ESLint 校验不通过则不允许进行源码的构建操作：

```javascript
"scripts": {
  "lint": "eslint src",
  "lint-strict": "eslint src --max-warnings 0",
  "build": "npm run lint && rimraf dist types && gulp",
}
```

需要注意在构建时进行校验的严格控制，一旦 lint 抛出 warning 或者 error 则立马终止构建（详情可查看 [ESLint 退出代码](https://cn.eslint.org/docs/user-guide/command-line-interface#exit-codes)）。

> **温馨提示**：需要注意 Shell 中的 `&&` 和 `&` 是有差异的，`&&` 主要用于继发执行，只有前一个任务执行成功，才会执行下一个任务，`&` 主要用于并发执行，表示两个脚本同时执行。这里构建的命令需要等待 `lint` 命令执行通过才能进行，一旦 `lint` 失败那么构建命令将不再执行。

#### ESLint 确保代码上传

尽管可能配置了 ESLint 的校验脚本 以及 VS Code 插件，但是有些 ESLint 的规则校验是无法通过 Save Auto Fix 进行格式化修复的（例如质量规则），因此还需要一层保障能够确保代码提交之前所有的代码能够通过 ESLint 校验，这个配置将在 Lint Staged 中进行讲解。

#### 文档

- [Npm 官方文档](https://docs.npmjs.com/)
- [使用 NPM 发布和使用 CLI 工具](https://juejin.im/post/5eb89053e51d454de54db501)
- [Top 10 JavaScript errors from 1000+ projects (and how to avoid them)](https://rollbar.com/blog/top-10-javascript-errors/)
- [Cz 工具集使用介绍](https://juejin.im/post/5cc4694a6fb9a03238106eb9)（强烈推荐阅读）
- [TypeScript 中文网](https://www.tslang.cn/)
- [tsconfig.json 编译选项](https://www.tslang.cn/docs/handbook/compiler-options.html)
- [gulp-typescript](https://github.com/ivogabe/gulp-typescript)
- [ES modules: A cartoon deep-dive](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)（强烈推荐阅读）
- [ESLint 中文网](https://cn.eslint.org/)
- [typescript-eslint](https://github.com/typescript-eslint/typescript-eslint)
- [Getting Started - Linting your TypeScript Codebase](https://github.com/typescript-eslint/typescript-eslint/blob/master/docs/getting-started/linting/README.md)

### Prettier

#### Prettier 背景

Prettier 是一个统一代码格式风格的工具，如果你不清楚为什么需要使用 Prettier，可以查看 [Why Prettier?](https://prettier.io/docs/en/why-prettier.html)。很多人可能疑惑，ESLint 已经能够规范我们的代码风格，为什么还需要 Prettier？在 [Prettier vs Linters](https://prettier.io/docs/en/comparison.html) 中详细说明了两者的区别，Linters 有两种类型的规则：

- 格式规则（Formatting rules）：例如 [max-len](https://eslint.org/docs/rules/max-len)、[keyword-spacing](https://eslint.org/docs/rules/keyword-spacing) 以及 [no-mixed-spaces-and-tabs](https://eslint.org/docs/rules/no-mixed-spaces-and-tabs) 等
- 质量规则（Code-quality rules）：例如 [no-unused-vars](https://eslint.org/docs/rules/no-unused-vars)、[no-implicit-globals](https://eslint.org/docs/rules/no-implicit-globals) 以及 [prefer-promise-reject-errors](https://eslint.org/docs/rules/prefer-promise-reject-errors) 等

ESLint 的规则校验同时包含了 **格式规则** 和 **质量规则**，但是大部分情况下只有 **格式规则** 可以通过 `--fix` 或 VS Code 插件的 Sava Auto Fix 功能一键修复，而 **质量规则** 更多的是发现代码可能出现的 Bug 从而防止代码出错，这类规则往往需要手动修复。因此 **格式规则** 并不是必须的，而 **质量规则** 则是必须的。Prettier 与 ESLint 的区别在于 Prettier 专注于统一的**格式规则**，从而减轻 ESLint 在**格式规则上**的校验，而对于**质量规则** 则交给专业的 ESLint 进行处理。总结一句话就是：Prettier for formatting and linters for catching bugs!（ESLint 是必须的，Prettier 是可选的！）

需要注意如果 ESLint（TSLint） 和 Prettier 配合使用时**格式规则**有重复且产生了冲突，那么在编辑器中使用 Sava Auto Fix 时会让你的一键格式化哭笑不得。此时应该让两者把各自注重的规则功能区分开，使用 ESLint 校验**质量规则**，使用 Prettier 校验**格式规则**，更多信息可查看 [Integrating with Linters](https://prettier.io/docs/en/integrating-with-linters.html)。

> **温馨提示**：在 VS Code 中使用 ESLint 匹配到相应的规则时会产生黄色波浪线以及红色文件名进行错误提醒。Prettier 更希望你对格式规则无感知，从而不会让你觉得有任何使用的负担。如果想要了解更多 Prettier，还可以查看 Prettier 的背后思想 [Option Philosophy](https://prettier.io/docs/en/option-philosophy.html)，个人认为了解一个产品设计的**哲学**能更好的指导你使用该产品。

#### Prettier 配置

首先安装 Prettier 所需要的依赖：

```javascript
npm i  prettier eslint-config-prettier --save-dev
```

其中：

- [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier): 用于解决 ESLint 和 Prettier 配合使用时容易产生的**格式规则**冲突问题，其作用就是关闭 ESLint 中配置的一些格式规则，除此之外还包括关闭 `@typescript-eslint/eslint-plugin`、`eslint-plugin-babel`、`eslint-plugin-react`、`eslint-plugin-vue`、`eslint-plugin-standard` 等格式规则。

理论上而言，在项目中开启 ESLint 的 `extends` 中设置的带有格式规则校验的规则集，那么就需要通过 `eslint-config-prettier` 插件关闭可能产生冲突的格式规则：

```javascript
{
  "extends": [
    "plugin:@typescript-eslint/recommended",
    // 用于关闭 ESLint 相关的格式规则集，具体可查看 https://github.com/prettier/eslint-config-prettier/blob/master/index.js
    "prettier",
    // 用于关闭 @typescript-eslint/eslint-plugin 插件相关的格式规则集，具体可查看 https://github.com/prettier/eslint-config-prettier/blob/master/%40typescript-eslint.js
    "prettier/@typescript-eslint",
  ]
}
```

配置完成后，可以通过[命令行接口](https://prettier.io/docs/en/cli.html)运行 Prettier:

```javascript
"scripts": {
  "prettier": "prettier src test --write",
},
```

`--write` 参数类似于 ESLint 中的 `--fix`（在 ESLint 中使用该参数还是需要谨慎哈，建议还是使用 VS Code 的 Save Auto Fix 功能），主要用于自动修复格式错误。此时书写格式的错误代码：

```javascript
import great from "@/greet";

// 中间加很多空行
export default {
  great,
};
```

执行 `npm run prettier` 进行格式修复：

```javascript
PS C:\Code\Git\algorithms> npm run prettier

> algorithms-utils@1.0.0 prettier C:\Code\Git\algorithms
> prettier src test --write

src\greet.ts 149ms
src\index.ts 5ms
test\greet.spec.ts 11ms
```

修复之后的的文件格式如下：

```javascript
import great from "@/greet";

export default {
  great,
};
```

需要注意如果某些规则集没有对应的 `eslint-config-prettier` 关闭配置，那么可以先通过 [CLI helper tool](https://github.com/prettier/eslint-config-prettier#cli-helper-tool) 检测是否有重复的格式规则集在生效，然后可以通过手动配置 `eslintrc.js` 的形式进行关闭：

```javascript
PS C:\Code\Git\algorithms> npx eslint --print-config src/index.ts | npx eslint-config-prettier-check
No rules that are unnecessary or conflict with Prettier were found.
```

例如把 `eslint-config-prettier` 的配置去除，此时进行检查重复规则：

```javascript
PS C:\Code\Git\algorithms> npx eslint --print-config src/index.ts | npx eslint-config-prettier-check
The following rules are unnecessary or might conflict with Prettier:

- @typescript-eslint/no-extra-semi
- no-mixed-spaces-and-tabs

The following rules are enabled but cannot be automatically checked. See:
https://github.com/prettier/eslint-config-prettier#special-rules

- no-unexpected-multiline
```

此时假设 `eslint-config-prettier` 没有类似的关闭格式规则集（例如本项目中配置的 `plugin:jest/recommended` 可能存在规则冲突），那么可以通过配置 `.eslintrc.js` 的形式自己手动关闭相应冲突的格式规则。

> **温馨提示**：ESLint 可以对不同的文件支持不同的规则校验， 因此 `--print-config` 只能对应单个文件的冲突格式规则检查。 由于通常的项目是一套规则对应一整个项目，因此对于整个项目所有的规则只需要校验一个文件是否有格式规则冲突即可。

#### Prettier 插件

通过命令行接口 `--write` 的形式可以进行格式自动修复，但是类似 ESLint，我们更希望项目在实时编辑时可以通过保存就能自动格式化代码（鬼知道 `--fix` 以及 `--write` 格式了什么文件，当然更希望通过肉眼的形式立即感知代码的格式化变化），此时可以通过配置 VS Code 的 [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) 插件进行 Save Auto Fix，具体的配置查看插件文档。

#### Prettier 确保代码上传

和 ESLint 一样，尽管可能配置了 Prettier 的自动修复格式脚本以及 VS Code 插件，但是无法确保格式遗漏的情况，因此还需要一层保障能够确保代码提交之前能够进行 Prettier 格式化，这个配置将在 Lint Staged 中讲解，更多配置方案也可以查看 [Prettier - Pre-commit Hook](https://prettier.io/docs/en/precommit.html)。

### Lint Staged

#### Lint Staged 背景

在 Git Commit Message 中使用了 [commitlint](https://commitlint.js.org/#/) 工具配合 husky 可以防止生成不规范的 Git Commit Message，从而阻止用户进行不规范的 Git 代码提交，其原理就是监听了 Git Hook 的执行脚本（会在特定的 Git 执行命令诸如 `commit`、`push`、`merge` 等触发之前或之后执行相应的脚本钩子）。Git Hook 其实是进行项目约束非常好用的工具，它的作用包括但不限于：

- Git Commit Message 规范强制统一
- ESLint 规则统一，防止不符合规范的代码提交
- Prettier 自动格式化（类似的还包括 Style 样式格式等）
- 代码稳定性提交，提交之前确保测试用例全部通过
- 发送邮件通知
- CI 集成（服务端钩子）

Git Hook 的钩子非常多，但是在客户端中可能常用的钩子是以下两个：

- `pre-commit`：Git 中 `pre` 系列钩子允许终止即将发生的 Git 操作，而`post` 系列往往用作通知行为。`pre-commit` 钩子在键入提交信息（运行 `git commit` 或 `git cz`）前运行，主要用于检查当前即将被提交的代码快照，例如提交遗漏、测试用例以及代码等。该钩子如果以非零值退出则 Git 将放弃本次提交。当然你也可以通过配置命令行参数 `git commit --no-verify` 绕过钩子的运行。
- `commit-msg`：该钩子在用户输入 Commit Message 后被调用，接收存有当前 **Commit Message** 信息的临时文件路径作为唯一参数，因此可以利用该钩子来核对 Commit Meesage 信息（在 Git Commit Message 中使用了该钩子对提交信息进行了是否符合 Angular 规范的校验）。该钩子和 `pre-commit` 类似，一旦以非零值退出 Git 将放弃本次提交。

除了上述常用的客户端钩子，还有两个常用的服务端钩子：

- `pre-receive`：该钩子会在远程仓库接收 `git push` 推送的代码时执行（注意不是本地仓库），该钩子会比 `pre-commit` 更加有约束力（总会有这样或那样的开发人员不喜欢提交代码时所做的一堆检测，他们可能会选择绕过这些钩子）。`pre-receive` 钩子可用于接收代码时的强制规范校验，如果某个开发人员采用了绕过 `pre-commit` 钩子的方式提交了一堆 💩 一样的代码，那么通过设置该钩子可以拒绝代码提交。当然该钩子最常用的操作还是用于检查是否有权限推送代码、非快速向前合并等。
- `post-receive`：该钩子在推送代码成功后执行，适合用于发送邮件通知或者触发 CI 。

> **温馨提示**：想了解更多 Git Hook 信息可以查看 [Git Hook 官方文档](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90) 或 [Git 钩子：自定义你的工作流](https://github.com/geeeeeeeeek/git-recipes/wiki/5.4-Git-%E9%92%A9%E5%AD%90%EF%BC%9A%E8%87%AA%E5%AE%9A%E4%B9%89%E4%BD%A0%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81)。

需要注意初始化 Git 之后默认会在 `.git/hooks` 目录下生成所有 Git 钩子的 Shell 示例脚本，这些脚本是可以被定制化的。对于前端开发而言去更改这些示例脚本适配前端项目非常不友好（大多数前端开发同学压根不会设计 Shell 脚本，尽管这个对于制作工具是一件非常高效的事情），因此社区就出现了类似的增强工具，它们对外抛出的是简单的钩子配置（例如 [ghooks](https://github.com/ghooks-org/ghooks) 在 `package.json` 中只需要进行简单的[钩子属性配置](https://github.com/ghooks-org/ghooks#setup)），而在内部则通过替换 Git 钩子示例脚本的形式使得外部配置的钩子可以被执行，例如 [husky](https://github.com/typicode/husky)、ghooks 以及 [pre-commit](https://github.com/pre-commit/pre-commit) 等。

> **温馨提示**： Git Hook 还可以定制脚本执行的语言环境，例如对于前端而言当然希望使用熟悉的 Node 进行脚本设计，此时可以通过在脚本文件的头部设置 `#! /usr/bin/env node` 将 Node 作为可执行文件的环境解释器，如果你之前看过 [使用 NPM 发布和使用 CLI 工具](https://juejin.im/post/5eb89053e51d454de54db501) 可能会对这个环境解析器相对熟悉，这里也给出一个使用 Node 解释器的示例：[ghooks - hook.template.raw](https://github.com/ghooks-org/ghooks/blob/master/lib/hook.template.raw)，ghooks 的实现非常简单，感兴趣的同学可以仔细阅读一些源码的实现。

介绍 Git Hook 是为了让大家清晰的认知到使用 Hook 可以在前端的工程化项目中做很多事情（本来应该放在 Git Commit Message 中介绍相对合适，但是鉴于那个小节引用了另外一篇文章，因此将这个信息放在本小节进行科普）。

之前提到使用 Git Hook 可以进行 ESLint 规范约束，因此大家其实应该能够猜到使用 `pre-commit` 钩子（当然需要借助 Git Hook 增强工具，本项目中一律选择 `husky`）配合 ESLint 可以进行提交说明前的项目代码规则校验，但是如果项目越来越大，ESLint 校验的时间可能越来越长，这对于频繁的代码提交者而言可能是一件相对痛苦的事情，因此可以借助 `lint-staged` 工具（听这个工具的名字就能够猜测 lint 的是已经放入 Git Stage 暂存区中的代码，`ed` 在英文中表明已经做过）减少代码的检测量。

#### Lint Staged 配置

使用 [commitlint](https://commitlint.js.org/#/) 工具可以防止生成不规范的 Git Commit Message，从而阻止用户进行 Git 代码提交。但是如果想要防止团队协作时开发者提交不符合 ESLint 规则的代码则可以通过 [lint-staged](https://github.com/okonet/lint-staged) 工具来实现。`lint-staged` 可以在用户提交代码之前（生成 Git Commit Message 信息之前）使用 ESLint 检查 Git 暂存区中的代码信息（`git add` 之后的修改代码），一旦存在 💩 一样不符合校验规则的代码，则可以终止提交行为。需要注意的是 `lint-staged` 不会检查项目的全量代码（全量使用 ESLint 校验对于较大的项目可能会是一个相对耗时的过程），而只会检查添加到 Git 暂存区中的代码。根据官方文档执行以下命令自动生成配置项信息：

```javascript
npx mrm lint-staged
```

需要注意默认生成的配置文件是针对 JavaScript 环境的，手动修改 `package.json` 中的配置信息进行 TypeScript 适配：


```javascript
// 我们的哈士奇再次上场，这次它是要咬着你的 ESLint 不放了，这里我简称它的动作为 "咬 💩" ~~~
"husky": {
  "hooks": {
    "pre-commit": "lint-staged"
  }
},
"lint-staged": {
  // 这里需要注意 ESLint 脚本的 --max-warnings 0
  // 否则就算存在 warning 也不会终止提交行为
  // 这里追加了 Prettier 的自动格式化，确保代码提交之前所有的格式能够修复
  "*.ts": "npm run lint-strict"
}
```

此时如果将要提交的代码有 💩 , 则提交时会提示错误信息且提交会被强制终止：

husky 在 `package.json` 中配置了 `pre-commit` 和 `commit-msg` 两个 [Git 钩子](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)，优先使用 `pre-commit` 钩子执行 ESLint 校验，如果校验失败则终止运行。如果校验成功则会继续执行 `commit-msg` 校验 Git Commit Message。

