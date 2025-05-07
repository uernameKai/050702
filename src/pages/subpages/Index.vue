<template>
  首页
</template>

<style lang="scss" scoped>
.el-row {
  margin-bottom: 20px;
  &:last-child {
    margin-bottom: 0;
  }
}

.el-col .box-card {
  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .info {
    font-size: 14px;
  }
}

.card-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border: 1px solid #e4e7ed;
  text-align: center;
  padding-right: 20px;

  .card-left-container {
    color: white;
  }

  .card-right-container .number {
    font-size: 18px;
    font-weight: bold;
  }
}
</style>