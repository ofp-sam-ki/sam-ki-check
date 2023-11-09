<script setup>

</script>


<template>
        
        
            <h1 style="margin:2vw;">SAM-KI-Check</h1>


<table class="start-table">
  <tr>
    <td class="start-table-cell" rowspan="2">Symbol</td>
    <td class="start-table-cell"><button style="width:90%">Neue Prüfung</button></td>
  </tr>
  <tr>    
    <td class="start-table-cell"><button style="width:90%">Prüfung fortsetzen</button></td>
  </tr>
</table>

</template>


<style scoped>
.start {
    display: grid;
    grid-template-columns: auto auto;
    grid-template-rows: auto auto;
    text-align: center;
    align-items: center;
}

h1 {
    font-size: 36px;
}
.centered {
    margin: auto;
    width: 300px;
    border: 3px solid green;
    padding: 10px;
}
.symbol-img {
    width: 80px;
    height: 80px;
}

.start-table {
    width: 50vw;
    border: 1px solid red;
    height: 50vh;
}

.start-table-cell {
    width: 25vw;
}

</style>
