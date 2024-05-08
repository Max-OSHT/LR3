<template ref="GetLog">
  
</template>

<script>
import axios from 'axios';
import myMixin from '../myMixin';

export default {
  mixins: [myMixin],
  data() {
    return {
      isAuthorized: true
    }
  },
  methods: {
    // logo(){
    //   this.$refs.GetLog.logo();
    // },
    // info(){
    //   this.$refs.GetLog.info();
    // },
    // warning(){
    //   this.$store.dispatch('warning');
    // },
    // error(){
    //   this.$refs.GetLog.error();
    // },
    // critical(){
    //   this.$refs.GetLog.critical();
    // },

    async logout() {
      this.$store.dispatch('logout')
      .then(() =>{this.$router.push({name:'home'})})
    }
  },
  computed: {
    isLoggedIn : function(){ return this.$store.getters.isLoggedIn}  
  },
  created() {
    axios.interceptors.response.use(undefined, function (err) {
      return new Promise((resolve, reject) => {
        if (err.status === 401 && err.config && !err.config.__isRetryRequest) {
          this.$store.dispatch('logout')
          .then(() =>{this.$router.push({name:'home'})})
        }
        throw err;
      });
    });
  }
}
</script>

<style lang="scss" scoped>

</style>
