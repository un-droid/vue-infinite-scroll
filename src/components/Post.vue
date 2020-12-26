<template>
    <div class="post">
        <b>Post id: {{id}}</b>
        <div class="body">Post body: {{body}}</div>
    </div>
</template>
<script>
export default {
    name: "post",
    props:{
        userId: Number,
        id: Number,
        title: String,
        body: String
    },
    data() {
        return {}
  },
}
</script>
<style scoped>
    .post{
        min-height: 100px;
        width: 400px;
        padding: 5px 2px;
        border: 1px solid black;
        border-radius: 3px;
        margin: 0 auto;
    }
    .body{
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }

</style>