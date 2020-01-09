<template>
  <div class="Header">
    <router-link to="/">
       <div class="header">
          <h1>Solar Logger</h1>          
       </div> 
    </router-link>
  </div>
</template>

<script>
  export default {

  }
</script>

<style lang="scss">
  @import "../assets/styles/_variables.scss";
  
.header {
  @include container();  
  font-family: Tahoma, Geneva, sans-serif;
  padding: 10px;  
  text-align: center;
  background: $header-backg-color;
  color: white;
  font-size: 20px;
}

a:hover, a:visited, a:link, a:active
{
    text-decoration: none;
}
</style>
