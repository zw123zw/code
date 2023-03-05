<template>
  <div class="goTop">
    <el-alert
      title="成功提示的文案"
      type="success"
      show-icon
      effect="light"
    ></el-alert>
    <el-autocomplete
      class="inline-input"
      v-model="state1"
      :fetch-suggestions="querySearch"
      placeholder="请输入内容"
      :popperAppendToBody="false"
    >
      <template slot="prepend">
        wewqe
      </template>
    </el-autocomplete>
    <el-badge :value="12" class="item" isDot>
      <el-button size="small">评论</el-button>
    </el-badge>
    <el-badge :value="122" :max="99" class="item" type="primary">
      <el-button size="small">评论</el-button>
    </el-badge>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item><a href="/">活动管理</a></el-breadcrumb-item>
      <el-breadcrumb-item>活动详情</el-breadcrumb-item>
    </el-breadcrumb>
    <el-breadcrumb separator-class="separator-test">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item><a href="/">活动管理</a></el-breadcrumb-item>
      <el-breadcrumb-item>活动详情</el-breadcrumb-item>
    </el-breadcrumb>
    <el-button type="primary" size="mini">主要按钮</el-button>
    <el-button-group>
      <el-button type="primary" icon="el-icon-edit"></el-button>
      <el-button type="primary" icon="el-icon-share"></el-button>
      <el-button type="primary" icon="el-icon-delete"></el-button>
    </el-button-group>
    <el-carousel
      height="200px"
      style="background: #9e60e2;overflow: hidden;"
      type="card"
    >
      <el-carousel-item v-for="item in 4" :key="item" :label="item">
        <h3 class="small">{{ item }}</h3>
      </el-carousel-item>
    </el-carousel>
    <el-cascader v-model="value" :options="options" :props="{ multiple: true }" filterable size="small"></el-cascader>

    <el-calendar v-model="dateValue"></el-calendar>
    <el-avatar :src="circleUrl" :size="100" icon="el-icon-user"></el-avatar>
    <el-avatar :src="circleUrl" shape="square" size="small"></el-avatar>
    <el-backtop target=".goTop" :visibilityHeight="100"></el-backtop>
  </div>
</template>

<script>
export default {
  data() {
    return {
      state1: '',
      dateValue: '',
      circleUrl:
        'https://cube.elemecdn.com/3/7c/3ea6beec64369c2642b92c6726f1epng.png',
      value: [],
      options: [
        {
          value: 'zhinan',
          label: '指南',
          children: [
            {
              value: 'shejiyuanze',
              label: '设计原则',
              children: [
                {
                  value: 'yizhi',
                  label: '一致',
                },
              ],
            },
          ],
        },
      ],
    }
  },
  methods: {
    querySearch(queryString, cb) {
      cb([
        { value: '三全鲜食（北新泾店）', address: '长宁区新渔路144号' },
        {
          value: 'Hot honey 首尔炸鸡（仙霞路）',
          address: '上海市长宁区淞虹路661号',
        },
      ])
    },
  },
}
</script>
<style lang="scss" scoped>
.goTop {
  height: 95vh;
  overflow: auto;
  overflow-x: hidden;

  div {
    margin: 10px;
  }

  .el-avatar {
    display: block;
    margin-top: 60vh;
  }

  ::v-deep .separator-test {
    display: inline-block;
    width: 10px;
    height: 10px;
    background: #c0c4cc;
  }
}
</style>
