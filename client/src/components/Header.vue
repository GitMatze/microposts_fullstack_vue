<template>
  <div class="Header">
       <div class="header">
          <h1>Solar Logger</h1>          
       </div> 
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
  padding: rem(1);
  text-align: center;
  background: $header-backg-color;
  color: white;
  font-size: rem(17);
}
@media (max-width: 600px) {
  .header {
    font-size: rem(10);    
  }
}

a:hover, a:visited, a:link, a:active
{
    text-decoration: none;
}
</style>
