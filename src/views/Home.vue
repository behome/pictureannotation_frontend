<template>
  <div class="home">
    <el-container direction="vertical" style="width: 100%; padding: 0; margin: 0;">
     <Header/>
    </el-container>
    <el-container>
      <el-main >
        <MainContent/>
      </el-main>
    </el-container>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import Header from '@/components/Header.vue'
import MainContent from '@/components/MainContent.vue'

export default {
  name: 'home',
  components: {
    HelloWorld,
    Header,
    MainContent
  }
}
</script>
