<template>
    <div class="header">
        <el-header style="width: 100%;" height="40px">
            <el-menu default-active="1" mode="horizontal" background-color="#fff" style="height: 40px; line-height: 40px;">
                <el-menu-item index="1" style="margin-left: 100px; height: 40px; line-height: 40px;">
                    <span slot="title">首页</span>
                </el-menu-item>
            </el-menu>
        </el-header>
    </div>

</template>
<script>
    export default {
        name: "Header"
    }
</script>

<style scoped>
.el-header {
    color: #333;
    text-align: center;
    line-height: 40px;
}
</style>
