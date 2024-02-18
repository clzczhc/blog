---
title: Vue 按条件添加类
date: 2022-02-13 19:08:12
categories: "前端"
tags:
	- Vue
---

# Vue 按条件添加类

```vue
<el-card class="box-card">
      <div class="question" ref="question" v-for="(question, index) in questionList" :key="question.id">
        <h4>{{(index+1) +10*(currentPage-1)}}. {{question.ques}}</h4>
        <div v-for="option in question.options" :key="option.id" :class="[
                                                                    {'choose': true},
                                                                    {'wrong': option.value[0] === question.useranswer}, 
                                                                    {'right': option.value[0] === question.answer}
                                                                  ]">
          <label>
            <span>{{option.value}}</span>
          </label>
        </div>
      </div>
      <el-pagination background layout="prev, pager, next, jumper" :total="total" :page-size="pageLimit" :hide-on-single-page="noPagination" @current-change="currentChange">
      </el-pagination>
    </el-card>
```

项目中直接抽出来的(人菜勿喷)

**关键**：

```vue
<div :class="[{ red: true }, { blue: false }]">红色</div>

<div :class="[{ red: false }, { blue: true }]">蓝色</div>
```

**css**：

```css
.red {
  color: red;
}

.blue {
  color: blue;
}
```

![image-20220213191710083](https://s2.loli.net/2022/02/13/G1XUM8Z2fgPj3uV.png)
