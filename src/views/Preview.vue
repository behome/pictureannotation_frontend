<template xmlns:v-contextmenu="http://www.w3.org/1999/xhtml">
    <div style="width: 100%; height: 800px; text-align: center;">
        <div style="width: 100%; text-align: center; position: relative; "
             ref="imageDom"
             v-contextmenu:contextmenu>
            <div :style="{
                border: 'black solid 1px',
                backgroundColor: 'white',
                backgroundRepeat:'no-repeat',
                backgroundPosition:'center',
                backgroundSize: 'contain',
                width: picWidth + 'px',
                height: picHeight + 'px',
                zIndex: '0'}"
                 class="imgArea"
                 >
            </div>
            <div class="box" v-for="(boxItem, index) in charBoxList"
                 >
                <div class="box-content"
                     :data-key="index"
                     style="color: red;"
                     :style="{width: boxItem.offsetX2 - boxItem.offsetX1 + 'px', height: boxItem.offsetY2 - boxItem.offsetY1 + 'px', top: boxItem.offsetY1 + 'px', left: boxItem.offsetX1 + 'px'} ">
                    {{boxItem.charStr}}
                </div>
            </div>
        </div>
        <v-contextmenu ref="contextmenu">
            <v-contextmenu-item @click="downLoadPicture">保存成图片</v-contextmenu-item>
        </v-contextmenu>
    </div>
</template>

<script>
    import html2canvas from "html2canvas"
    export default {
        name: "Preview",
        data(){
            return {
                charBoxList:[],
                picWidth: 0,
                picHeight: 0,
                imgUrl: ''
            }
        },
        mounted(){
            this.loadInfo();
        },
        methods:{
            loadInfo(){
                this.charBoxList = JSON.parse(localStorage.getItem("charList"));
                this.picHeight = localStorage.getItem("picHeight");
                this.picWidth = localStorage.getItem("picWidth");
            },
            downLoadPicture(){
                html2canvas(this.$refs.imageDom).then(canvas => {
                    // 转成图片，生成图片地址
                    this.imgUrl = canvas.toDataURL("image/png");
                    var eleLink = document.createElement("a");
                    eleLink.href = this.imgUrl; // 转换后的图片地址
                    eleLink.download = "preview.png";
                    // 触发点击
                    document.body.appendChild(eleLink);
                    eleLink.click();
                    // 然后移除
                    document.body.removeChild(eleLink);
                    console.log(this.imgUrl);
                });
            }
        }
    }
</script>


<style scoped>
    .box-content{
        border: chartreuse solid 1px;
        position: absolute;
        z-index: 999;
        cursor: move;
    }

</style>

