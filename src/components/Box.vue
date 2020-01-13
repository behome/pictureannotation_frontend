<template>
    <div class="box" >
        <div class="box-point-1">

        </div>
        <div class="box-content" @click="choose()">

        </div>
        <div class="box-point-2">

        </div>
    </div>
</template>

<script>
    export default {
        props:['info'],
        name: "Box",
        data(){
            return {

            }
        },
        methods:{
            choose(){
                console.log()
            }
        }
    }
</script>

<style scoped>
    .box-point-1{
        width: 6px;
        height: 6px;
        position: absolute;
        top: 700px;
        left: 1000px;
        background-color: #ffff00;
        z-index: 9999;
    }
    .box-point-2{
        width: 6px;
        height: 6px;
        position: absolute;
        top: 721px;
        left: 1021px;
        background-color: #ffff00;
        z-index: 9999;
    }
    .box-content{
        border: chartreuse solid 1px;
        width: 20px;
        height: 20px;
        position: absolute;
        top: 703px;
        left: 1003px;
        z-index: 999;
        cursor: move;
    }
</style>
