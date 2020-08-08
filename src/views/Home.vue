<template>
  <div class="home">
    <NavBar />
  <MessageList />
<!--  <img alt="Vue logo" src="../assets/logo.png">
  <HelloWorld msg="Welcome to Your Vue.js App"/>-->
</div>
</template>

<script>
// @ is an alias to /src
//import HelloWorld from '@/components/HelloWorld.vue'
import NavBar from '@/components/NavBar.vue'
import MessageList from '@/components/MessageList.vue'


export default {
  name: 'Home',
  components: {
  //  HelloWorld,
    NavBar,
    MessageList
  }
}
</script>
