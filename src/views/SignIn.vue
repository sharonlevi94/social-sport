<template>
  <div class="signnin">
    <JoinUs @addSomeThing="reloadTable()"/>
  </div>
</template>

<script>
// @ is an alias to /src
import JoinUs from "../components/JoinUs";

export default {
  name: 'SignIn',
  components: {
    JoinUs
  },
  data(){
    return{
      isReload: false,
    }
  },
  methods: {
    reloadTable(){
      this.isReload = !this.isReload;
    },
  }
}
</script>
