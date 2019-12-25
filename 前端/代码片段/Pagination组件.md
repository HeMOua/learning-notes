# Pagination组件

## 组件

在`components`文件夹创建`index.vue`文件，内容如下

```vue
<template>
  <div :class="{'hidden': hidden}" class="pagination-container">
    <el-pagination
      :background="background"
      :current-page.sync="currentPage"
      :page-size.sync="pageSize"
      :page-sizes="pageSizes"
      :layout="layout"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>

<script>
export default {
  props: {
    total: {
      type: Number,
      require: true,
      default: 0
    },
    page: {
      type: Number,
      default: 1
    },
    limit: {
      type: Number,
      default: 10
    },
    pageSizes: {
      type: Array,
      default() {
        return [5, 10, 20, 30, 50]
      }
    },
    layout: {
      type: String,
      default: 'total, sizes, prev, pager, next, jumper'
    },
    background: {
      type: Boolean,
      default: true
    }
  },
  computed: {
    hidden() {
      return this.total <= 0
    },
    pageSize: {
      get() {
        return this.limit
      },
      set(val) {
        this.$emit('update:limit', val)
      }
    },
    currentPage: {
      get() {
        return this.page
      },
      set(val) {
        this.$emit('update:page', val)
      }
    }
  },
  methods: {
    handleSizeChange() {
      this.$emit('pagination')
    },
    handleCurrentChange() {
      this.$emit('pagination')
    }
  }
}
</script>

<style lang="scss" scoped>
.pagination-container {
  padding: 20px 0;
  &.hidden {
    display: none;
  }
  & > .el-pagination {
    float: right;
  }
  &::after {
   display: block;
   content: '';
   clear: both;
  }
}
</style>
```

## 使用

```vue
<pagination 
      :total="total"
      :page.sync="queryForm.pageNum"
      :limit.sync="queryForm.pageSize"
      @pagination="getList"
    />
```

## 结合scroll-to.js

```vue
<template>
  <div :class="{'hidden': hidden}" class="pagination-container">
    <el-pagination
      :background="background"
      :current-page.sync="currentPage"
      :page-size.sync="pageSize"
      :page-sizes="pageSizes"
      :layout="layout"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange">
    </el-pagination>
  </div>
</template>

<script>
import scrollTo from '@/utils/scroll-to'

export default {
  props: {
    total: {
      require: true,
      type: Number
    },
    page: {
      type: Number,
      default: 1
    },
    limit: {
      type: Number,
      default: 10
    },
    pageSizes: {
      type: Array,
      default() {
        return [5, 10, 20, 30, 50]
      }
    },
    layout: {
      type: String,
      default: 'total, sizes, prev, pager, next, jumper'
    },
    background: {
      type: Boolean,
      default: true
    },
    autoScroll: {
      type: Boolean,
      default: true
    }
  },
  computed: {
    hidden() {
      return this.total <= 0
    },
    pageSize: {
      get() {
        return this.limit
      },
      set(val) {
        this.$emit('update:limit', val)
      }
    },
    currentPage: {
      get() {
        return this.page
      },
      set(val) {
        this.$emit('update:page', val)
      }
    }
  },
  methods: {
    handleSizeChange() {
      this.$emit('pagination')
      if (this.autoScroll) {
        scrollTo(0, 800)
      }
    },
    handleCurrentChange() {
      this.$emit('pagination')
      if (this.autoScroll) {
        scrollTo(0, 800)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.pagination-container {
  padding: 20px 0;
  &.hidden {
    display: none;
  }
  & > .el-pagination {
    float: right;
  }
  &::after {
   display: block; 
   content: '';
   clear: both;
  }
}
</style>
```

