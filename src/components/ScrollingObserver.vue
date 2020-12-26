<template>
    <div class="observer"></div>
</template>
<script>
export default {
    name: "observer",
    date(){
        return {
            observer: null
        } 
    },
    mounted(){
        this.observer = new IntersectionObserver(([entry])=>{
            if(entry && entry.isIntersecting){
                this.$emit("intersect"); //emit the intersect event
            }
        });
        this.observer.observe(this.$el); //reference to the element which is inside the template
    },
    destroyed () {
        delete this.observer
    }
}
</script>
<style scoped>
    /* observer div should have a height */
    .observer{
        height: 1px;
    }
</style>