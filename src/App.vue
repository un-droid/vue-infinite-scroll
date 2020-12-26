<template>
  <div id="app">
    <div v-if="allPosts">Total {{allPosts.length}} posts loaded from server</div>
    <div v-if="visiblePosts">Presenting {{visiblePosts.length}} posts (2 pages ahead)</div>
    <div class="postsContainer" ref="postsContainerRef" v-if="visiblePosts">
      <Post v-for="post in visiblePosts" v-bind="post" v-bind:key="post.id"/>
      <ScrollingObserver v-on:intersect="observerIntersected"/>
    </div>
  </div>
</template>

<script>
import ScrollingObserver from "./components/ScrollingObserver";
import Post from "./components/Post";

export default {
  name: 'App',
  components: {
    ScrollingObserver,
    Post
  },
  data(){
  return {
    allPosts: [],
    visiblePosts: [],
    containerHeight: 0,
    //notice the following param, it makes the fetched data to contain two pages upfront
    requiredPagesNumber: 2,
    requiredPostHeight: 100,
    currentPostIndex: 0,
    pageSize: 0,
    pageIndex: 0,
    pagedPostsData: [],
    currentPage: 0,
  }
},
  created(){

  },
  mounted() {
    //get container size and prepare required data size (container height/100px)
    this.$nextTick(()=> {
    //assumption, posts container's height is a 3 digit number
    this.containerHeight = Math.floor(this.$refs.postsContainerRef.offsetHeight / 100) * 100;
  })
},
watch:{
  containerHeight(newVal, oldVal){
    if(newVal != oldVal){
      console.log(newVal);
      this.fetchDataFromServer().then(res=>{
        this.allPosts = [...res];
        this.calcPageSize();
        //convert all posts to a paged structure array
        this.pagedPostsData = this.chunkify(this.pageSize, this.allPosts);
        this.visiblePosts = this.pagedPostsData[0];
      });
    }
  }
},
  methods:{
    observerIntersected(){
      if(this.visiblePosts.length > 0){
        this.visiblePosts = this.getItems(++this.currentPage);
      }
    },
    getItems(pageIndex){
      //concat to visible posts a new chunk of data
      this.visiblePosts = [...JSON.parse(JSON.stringify(this.visiblePosts)), ...JSON.parse(JSON.stringify(this.pagedPostsData[pageIndex]))];
      return Array.from(this.visiblePosts);
    },
    calcPageSize(){
      /*calcPageSize is a seperate method for a case when we want to listen to container resize and
        recalculate the wanted posts amount.
        for example, container height is 523 pixels, round it down to 500. 500/100 is 5 posts
        the requirement is to load two pages, so this is the formula:
      */
      this.pageSize = (this.containerHeight / this.requiredPostHeight) * this.requiredPagesNumber;
    },
    chunkify(chunkSize, dataArr){
     //split the posts data into an array of pages
      return dataArr.reduce((resultArray, item, index) => {
        const chunkIndex = Math.floor(index/chunkSize)

        if(!resultArray[chunkIndex]) {
          resultArray[chunkIndex] = [] // start a new chunk
        }
        resultArray[chunkIndex].push(item)
        return resultArray;
      }, []);
    },
    async fetchDataFromServer(){

      return fetch(`https://jsonplaceholder.typicode.com/comments`).then(async response => {
        const data = await response.json();
        // check for error response
        if (!response.ok) {
          // get error message from body or default to response statusText
          const error = (data && data.message) || response.statusText;
          return Promise.reject(error);
        }
        return data;
      })
      .catch(error => {
        console.error("There was an error loading posts", error);
      });

    }
  }
}
</script>

<style>
#app {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
}
.postsContainer{
    height: 400px;
    width: 500px;
    overflow-y: auto;
    align-items: center;
    border: 1px solid lightgray;
    border-radius: 4px;
    padding: 15px;
}
</style>
