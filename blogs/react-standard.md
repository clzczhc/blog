---
title: React搭建规范
categories: 前端
date: 2023-10-03 22:44:44
tags:
    - 规范
---

## 创建项目

npx create-react-app react-standard-f --template typescript

## 1. prettier

> yarn add prettier --dev

.prettierrc

```js
{
  "printWidth": 100,
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "bracketSpacing": true,
  "arrowParens": "always",
  "proseWrap": "preserve"
}
```

## 2. Eslint

> npm init @eslint/config
> 选择yarn

tsconfig.json添加eslint配置文件

```json
"include": [
    "src",
    "./.eslintrc.js"
  ]
```

修改部分规则：
.eslintrc.js

```js
rules: {
    '@typescript-eslint/semi': 'off',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/space-before-function-paren': 'off',
    '@typescript-eslint/strict-boolean-expressions': 'off'
  }
```

## 3. EditorConfig 代码风格统一工具

.editorconfig

```python
# http://editorconfig.org
root = true

[*]
#缩进风格：空格
indent_style = space
#缩进大小2
indent_size = 2
#换行符lf
end_of_line = lf
#字符集utf-8
charset = utf-8
#是否删除行尾的空格
trim_trailing_whitespace = true
#是否在文件的最后插入一个空行
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false

[Makefile]
indent_style = tab
```

## 4. Stylelint

> yarn add stylelint stylelint-config-standard --dev

.stylelintrc.js

```js
module.exports = {
  extends: 'stylelint-config-standard',
  rules: {
    // your rules
  },
  // 忽略其他文件，只校验样式相关的文件
  ignoreFiles: [
    'node_modules/**/*',
    'public/**/*',
    'dist/**/*',
    '**/*.js',
    '**/*.jsx',
    '**/*.tsx',
    '**/*.ts',
  ],
};
```

添加脚本
package.json

```json
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "lint:style": "stylelint --fix \"src/**/*.{css,scss,less}\""
  },
```

## 5. husky 和 lint-staged

1. `npx husky-init`
   
   > - 安装 husky 到开发依赖
   > - 在项目根目录下创建 .husky 目录
   > - 在 .husky 目录创建 pre-commit hook，并初始化 pre-commit 命令为 npm test
   > - 修改 package.json 的 scripts，增加 "prepare": "husky install"

2. yarn

3. 修改 .husky/pre-commit hook 文件的触发命令：`eslint --fix ./src --ext .tsx,.ts,.js` 当我们`commit` 时，会先对 src 目录下所有的 .tsx、.ts、.js 文件执行 `eslint --fix`命令，如果 ESLint 通过，成功 commit，否则终止 commit

上面会出现，如果只改动一点点文件，提交也会对所有文件执行`eslint --fix`操作。所以可以使用`lint-staged`来将`husky`的触发变成只作用于git暂存区的文件（即`git add`的文件），而不会影响其他文件。

1. `yarn add lint-staged --dev`
2. package.json添加`lint-staged`配置

```json
"lint-staged": {
    "*.{tsx,ts,js}": "eslint --fix"
  }
```

3. 修改 .husky/pre-commit hook 的触发命令为：npx lint-staged

## 6. 提交规范

1. yarn add commitizen --dev

2. npx commitizen init cz-conventional-changelog --yarn --dev --exact
   
   > 安装 cz-conventional-changelog 到开发依赖（devDependencies）
   > 在 package.json 中增加了 config.commitizen

3. 更换为`git cz`提交代码（将`git cz`添加到`scripts`中后再执行） [![pPLArLDpng](https://z1.ax1x.com/2023/10/01/pPLArLD.png)](https://imgse.com/i/pPLArLD)

cz-customizable

1. `yarn add cz-customizable --dev`
2. 修改`package.json`

```json
"config": {
  "commitizen": {
    "path": "./node_modules/cz-customizable"
  }
}
```

3. 创建配置文件`.cz-config.js`。
   .cz-config.js

```js
module.exports = {
  // 提交类型
  types: [
    { value: 'feat', name: 'feat: 新增功能' },
    { value: 'fix', name: 'fix: 修复bug' },
    { value: 'docs', name: 'docs: 文档变更' },
    { value: 'style', name: 'style: 样式' },
    { value: 'refactor', name: 'refactor: 代码重构' },
    { value: 'perf', name: 'perf: 性能优化' },
    { value: 'test', name: 'test: 添加、修改测试用例' },
    { value: 'revert', name: 'revert:   回滚 commit' },
  ],

  // scope 类型（定义之后，可通过上下键选择）
  scopes: [
    ['components', '组件相关'],
    ['hooks', 'hook 相关'],
    ['utils', 'utils 相关'],
    ['styles', '样式相关'],
    ['other', '其他修改'],
    ['custom', '以上都不是？我要自定义'],
  ].map(([value, description]) => {
    return {
      value,
      name: `${value.padEnd(30)} (${description})`,
    };
  }),
  // 交互提示信息
  messages: {
    type: '选择你要提交的类型：',
    scope: '\n选择一个 scope（可选）：',
    // 选择 scope: custom 时会出下面的提示
    customScope: '请输入自定义的 scope：',
    subject: '填写简短精炼的变更描述：\n',
    body: '填写更加详细的变更描述（可选）。使用 "|" 换行：\n',
    breaking: '列举非兼容性重大的变更（可选）：\n',
    footer: '列举出所有变更的 ISSUES CLOSED（可选）。 例如: #31, #34：\n',
    confirmCommit: '确认提交？',
  },

  // 设置只有 type 选择了 feat 或 fix，才询问 breaking message
  allowBreakingChanges: ['feat', 'fix'],

  // 跳过要询问的步骤
  skipQuestions: ['body', 'breaking', 'footer'],

  // subject 限制长度
  subjectLimit: 100,
  breaklineChar: '|', // 支持 body 和 footer
  // footerPrefix : 'ISSUES CLOSED:'
  // askForBreakingChangeFirst : true,
};
```

[![pPLE7jKpng](https://z1.ax1x.com/2023/10/01/pPLE7jK.png)](https://imgse.com/i/pPLE7jK)

commitlint

1. `yarn add @commitlint/config-conventional @commitlint/cli --dev`

2. 创建`commit.config.js`文件

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
};
```

3. 使用`husky`命令在`.husky`目录创建`commit-msg`文件 `npx husky add .husky/commit-msg "npx --no-install commitlint --edit $1"`

tsconfig.json添加commit.config.js（不然会报错） [![pPLZRY9png](https://z1.ax1x.com/2023/10/01/pPLZRY9.png)](https://imgse.com/i/pPLZRY9)

```json
"include": [
    "src",
    "./.eslintrc.js",
    "./commitlint.config.js"
  ]
```

[![pPLZ7wDpng](https://z1.ax1x.com/2023/10/01/pPLZ7wD.png)](https://imgse.com/i/pPLZ7wD)
