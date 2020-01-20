<template>
    <div>
        <div style="width: 100%; height: 800px; text-align: center;">
            <div v-if="isLoad" style="width: 100%; height: 800px; text-align: center; position: relative;"
                 @mousemove="updateBox($event)"
            @mouseup="disableMove($event)">
                <div :style="{
                backgroundImage: 'url(' + picList[curImgIndex].imgUrl + ')',
                backgroundRepeat:'no-repeat',
                backgroundPosition:'center',
                backgroundSize: 'contain',
                width: picList[curImgIndex].width + 'px',
                height: picList[curImgIndex].height + 'px',
                zIndex: '0'}"
                     class="imgArea"
                     @mousedown="activateThis($event, 0)"
                    >
                </div>
                <div class="box" v-for="(boxItem, index) in picList[curImgIndex].boxes" v-if="boxItem.label === boxType"
                @contextmenu="deleteBox($event, index)">
                    <div v-if="index === picList[curImgIndex].activateBoxIndex"
                         class="box-point-1" :data-key="index"
                         :style="{left: boxItem.x1 - 3 + 'px', top: boxItem.y1 - 3 + 'px'}"
                         @mousedown="activateThis($event, index)">
                    </div>
                    <div class="box-content"
                         :data-key="index"
                         @mousedown="activateThis($event, index)"
                         :style="{width: boxItem.offsetX2 - boxItem.offsetX1 + 'px', height: boxItem.offsetY2 - boxItem.offsetY1 + 'px', top: boxItem.offsetY1 + 'px', left: boxItem.offsetX1 + 'px'} ">
                    </div>
                    <div v-if="index === picList[curImgIndex].activateBoxIndex"
                         class="box-point-2"
                         :data-key="index"
                         @mousedown="activateThis($event, index)"
                         :style="{left: boxItem.x2 - 3 + 'px', top: boxItem.y2 - 3 + 'px'}"
                         >

                    </div>
                </div>
            </div>
        </div>

        <div>

        </div>
        <div class="toolbox">
            <div>
                <el-select v-model="chosenDir" placeholder="请选择">
                    <el-option
                            v-for="item in dirs"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value">
                    </el-option>
                </el-select>
                <el-button type="primary" @click="getBoxInfo" size="medium" style="margin-left: 10px;">载入图片</el-button>
            </div>
            <el-button-group style="margin-top: 20px;">
                <el-button type="primary" icon="el-icon-arrow-left" @click="aboveImgIndex">上一张</el-button>
                <el-button type="primary" @click="nextImgIndex">下一张<i class="el-icon-arrow-right el-icon--right"></i></el-button>
                <el-button type="primary" @click="enableDisableAdd" icon="el-icon-plus">{{btnContent}}</el-button>
            </el-button-group>
            <div style="margin-top: 20px; width: 98%; background-color: white; border-radius: 5px; margin-left: auto; margin-right: auto;height: 30px;">
                <el-radio v-model="boxType" label="1" style="margin-top: 8px; margin-left: 20%;">行边框</el-radio>
                <el-radio v-model="boxType" label="2" style="margin-top: 8px;" @change="showBoxType">字边框</el-radio>
            </div>
            <div style="margin-top: 20px; ">
                <el-button type="primary" @click="saveThisPic">保存当前图片信息</el-button>
            </div>
        </div>
    </div>

</template>

<script>
    export default {
        name: "MainContent",
        data(){
            return{
                isLoad: false,
                dirs:[],
                chosenDir:'',
                boxType: '1',
                picList:[

                ],
                activeIndex: 0,
                curImgIndex: 0,
                isDown: false,
                addAnnotation: false,
                btnContent: "新增图框",
                enableMove: false,
                enableScalePoint1: false,
                enableScalePoint2: false,
                startX: 0,
                startY: 0
            }
        },
        mounted(){
          this.getDirs();
        },
        methods:{
            startDraw(e){
                e.preventDefault();
            },
            // 鼠标移动，更新矩形框信息
            updateBox(e){
                if(this.enableMove || this.enableScalePoint1 || this.enableScalePoint2){// 如果确认有激活的点或者矩形框内部，则计算鼠标偏移量
                    var index = this.picList[this.curImgIndex].activateBoxIndex;
                    var diffX = e.pageX - this.startX;
                    var diffY = e.pageY - this.startY;
                    this.startX = e.pageX;
                    this.startY = e.pageY;
                }
                if(this.enableMove){// 移动矩形框
                    this.picList[this.curImgIndex].boxes[index].offsetX1 += diffX;
                    this.picList[this.curImgIndex].boxes[index].offsetY1 += diffY;
                    this.picList[this.curImgIndex].boxes[index].offsetX2 += diffX;
                    this.picList[this.curImgIndex].boxes[index].offsetY2 += diffY;
                    this.picList[this.curImgIndex].boxes[index].x1 += diffX;
                    this.picList[this.curImgIndex].boxes[index].y1 += diffY;
                    this.picList[this.curImgIndex].boxes[index].x2 += diffX;
                    this.picList[this.curImgIndex].boxes[index].y2 += diffY;
                }else if(this.enableScalePoint1){// 左上角缩放
                    // this.picList[this.curImgIndex].boxes[index].offsetX = e.offsetX;
                    this.picList[this.curImgIndex].boxes[index].x1 += diffX;
                    this.picList[this.curImgIndex].boxes[index].y1 += diffY;
                    this.handleBox(index);
                }else if(this.enableScalePoint2){// 右下角缩放
                    this.picList[this.curImgIndex].boxes[index].x2 += diffX;
                    this.picList[this.curImgIndex].boxes[index].y2 += diffY;
                    this.handleBox(index);
                }
            },
            // 更新当前图片指定矩形框的右上角与左下角坐标
            handleBox(index){
                var box = this.picList[this.curImgIndex].boxes[index];
                // 由于移动缩放点的时候，缩放点不会保持原来左上右下的搭配，有可能原来左上操作点的x坐标跑到原来右下操作点的右边
                // 所以需要根据两个操作点的xy较小较大值，来更新实际的矩形框坐标
                this.picList[this.curImgIndex].boxes[index].offsetX1 = box.x1 > box.x2 ? box.x2 : box.x1;
                this.picList[this.curImgIndex].boxes[index].offsetY1 = box.y1 > box.y2 ? box.y2 : box.y1;
                this.picList[this.curImgIndex].boxes[index].offsetX2 = box.x1 > box.x2 ? box.x1 : box.x2;
                this.picList[this.curImgIndex].boxes[index].offsetY2 = box.y1 > box.y2 ? box.y1 : box.y2;
            },
            // 下一张图片
            nextImgIndex(){
                this.curImgIndex += 1;
                if(this.curImgIndex >= this.picList.length){
                    this.curImgIndex = this.picList.length - 1;
                    this.$message(
                        {
                            type:"warning",
                            message: "最后一张",
                            duration: 1500
                        }
                    )
                }
            },
            // 上一张图片
            aboveImgIndex(){
                this.curImgIndex -= 1;
                if(this.curImgIndex < 0){
                    this.curImgIndex = 0;
                    this.$message(
                        {
                            type:"warning",
                            message: "到顶了",
                            duration: 1500
                        }
                    )
                }
            },
            // 鼠标左键落下，激活当前矩形框，并根据点击位置，确认下一步动作
            activateThis(e, index){
                // 获取点击的位置
                this.startX = e.pageX;
                this.startY = e.pageY;

                if(e.target.className === 'box-content'){// 如果点击在矩形框内部，则下一步准备移动矩形框整体
                    this.enableMove = true;
                    this.picList[this.curImgIndex].activateBoxIndex = index;
                }else if(e.target.className === 'box-point-1'){// 如果点击在左上角点，则下一步准备移动左上角点，并缩放
                    this.enableScalePoint1 = true;
                    this.picList[this.curImgIndex].activateBoxIndex = index;
                }else if(e.target.className === 'box-point-2'){// 如果点击在右下角点，则下一步准备移动右下角点，并缩放
                    this.enableScalePoint2 = true;
                    this.picList[this.curImgIndex].activateBoxIndex = index;
                }else if(e.target.className === 'imgArea'){// 如果点击在图片区域，则该处没有矩形框，则需要判断是否增加矩形框
                    if(this.addAnnotation){// 确认添加矩形框，并激活
                        this.isDown = true;
                        var offsetX = e.offsetX;
                        var offsetY = e.offsetY;
                        this.picList[this.curImgIndex].boxes.push(
                            {
                                offsetX1: offsetX, // 矩形框左上角点x坐标
                                offsetY1: offsetY, // 矩形框左上角点y坐标
                                offsetX2: offsetX + 20, // 矩形框右下角点x坐标
                                offsetY2: offsetY + 20, // 矩形框右下角点y坐标
                                x1: offsetX, // 矩形框左上角缩放操作点x坐标
                                y1: offsetY, // 矩形框左上角缩放操作点y坐标
                                x2: offsetX + 20, // 矩形框右下角缩放操作点x坐标
                                y2: offsetY + 20, // 矩形框右下角缩放操作点y坐标
                                id: 0, // 保留字段id，还没使用
                                label: this.boxType // 矩形框类型，分为单字矩形框，与行矩形框
                            }
                        );
                        this.picList[this.curImgIndex].activateBoxIndex = this.picList[this.curImgIndex].boxes.length - 1;
                    }
                }
            },
            // 是否允许新建矩形框
            enableDisableAdd(){
                this.addAnnotation = !this.addAnnotation;
                if(this.addAnnotation){
                    this.btnContent = "退出"
                }else{
                    this.btnContent = "新增图框"
                }
            },
            moveAnnotation(e, index){
                // if(this.enableMove){
                //     this.picList[this.curImgIndex].boxes[index].offsetX += e.offsetX;
                //     this.picList[this.curImgIndex].boxes[index].offsetY += e.offsetY;
                //     console.log("jiji");
                // }
            },
            // 游标上抬取消所有移动
            disableMove(e){
                this.enableMove = false;
                this.enableScalePoint1 = false;
                this.enableScalePoint2 = false;
                this.isDown = false;
            },
            // 右键删除一矩形框
            deleteBox(e, index){
                e.preventDefault();
                this.picList[this.curImgIndex].boxes.splice(index, 1);
            },
            // 获取一个目录下的所有图片信息
            getBoxInfo(){
                if(this.chosenDir === undefined || this.chosenDir === ''){
                    this.$message(
                        {
                            type: 'warning',
                            message: "请先选择文件夹",
                            duration: 1500
                        }
                    )
                }else{
                    this.$ajax.get(
                        'http://192.168.1.112:8009/backend/getJson',
                        {
                            params:{
                                dirName: this.chosenDir
                            }
                        }
                    ).then(response => {
                        this.isLoad = false;
                        this.handleBoxInfo(response.data.data);
                    }).catch(err => {
                        console.log(err);
                    })
                }
            },
            // 处理有后端获取的矩形框的信息，转化为前端能够渲染的信息
            handleBoxInfo(data){
                this.picList = [];
                for(var i = 0, length = data.length; i < length; i++){
                    var picDict = {};
                    picDict['name'] = data[i].name;
                    picDict['imgUrl'] = 'http://192.168.1.112:8090/images/pic/' + this.chosenDir + '/' + data[i].name;
                    picDict['width'] = data[i].width;
                    picDict['height'] = data[i].height;
                    picDict['activateBoxIndex'] = 0;
                    var boxes = [];
                    for(var j = 0; j < data[i].wordsResult.length; j++){
                        boxes.push({
                            offsetX1: data[i].wordsResult[j].left,
                            offsetY1: data[i].wordsResult[j].top,
                            offsetX2: data[i].wordsResult[j].left + data[i].wordsResult[j].width,
                            offsetY2: data[i].wordsResult[j].top + data[i].wordsResult[j].height,
                            x1: data[i].wordsResult[j].left,
                            y1: data[i].wordsResult[j].top,
                            x2: data[i].wordsResult[j].left + data[i].wordsResult[j].width,
                            y2: data[i].wordsResult[j].top + data[i].wordsResult[j].height,
                            id: 0,
                            label: '1'
                        });
                        var charBoxes = data[i].wordsResult[j].simpleChar;
                        for(var k in charBoxes){
                            boxes.push({
                                offsetX1: charBoxes[k].left,
                                offsetY1: charBoxes[k].top,
                                offsetX2: charBoxes[k].left + charBoxes[k].width,
                                offsetY2: charBoxes[k].top + charBoxes[k].height,
                                x1: charBoxes[k].left,
                                y1: charBoxes[k].top,
                                x2: charBoxes[k].left + charBoxes[k].width,
                                y2: charBoxes[k].top + charBoxes[k].height,
                                id: 0,
                                label: '2'
                            });
                        }
                    }
                    picDict['boxes'] = boxes;
                    this.picList.push(picDict);
                }
                this.curImgIndex = 0;
                this.isLoad = true;
            },
            // 保存当前图片矩形框信息
            saveThisPic(){
                var form = new FormData();
                form.append("data", JSON.stringify(this.picList[this.curImgIndex]));
                form.append('dirName', this.chosenDir);
                this.$ajax.post(
                    'http://192.168.1.112:8009/backend/upload',
                    form
                ).then(res => {
                    if(res.data.flags === 'OK'){
                        this.$message(
                            {
                                type: 'success',
                                message: '保存成功',
                                duration: 1500
                            }
                        )
                    }else{
                        this.$message(
                            {
                                type: 'error',
                                message: '保存失败',
                                duration: 1500
                            }
                        )
                    }
                }).catch(err => {
                   console.log(err);
                });
            },
            // 获取后端图片目录
            getDirs(){
                this.$ajax.get(
                    'http://192.168.1.112:8009/backend/getDirs'
                ).then(response => {
                    for(var i = 0; i < response.data.data.length; i++){
                        this.dirs.push({
                            label: response.data.data[i],
                            value: response.data.data[i]
                        })
                    }
                }).catch(err => {
                    console.log(err);
                });
            },
            // 调试使用，显示当前矩形框类型的ratio按钮值
            showBoxType(){
              console.log(this.boxType);
            }
        }
    }
</script>

<style scoped>
    .box-point-1{
        width: 4px;
        height: 4px;
        position: absolute;
        background-color: #ffffff;
        border: #000 solid 1px;
        z-index: 9999;
    }
    .box-point-2{
        width: 4px;
        height: 4px;
        position: absolute;
        background-color: #ffffff;
        border: #000 solid 1px;
        z-index: 9999;
    }
    .box-content{
        border: chartreuse solid 1px;
        position: absolute;
        z-index: 999;
        cursor: move;
    }
    .active{
        display: block;
    }
    .toolbox{
        position: fixed;
        right: 100px;
        top: 90px;
        border: #2c3e50 solid 1px;
        padding: 5px;
        border-radius: 5px;
        background-color: dimgray;
    }
</style>
