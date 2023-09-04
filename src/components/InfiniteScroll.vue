<template>
  <div class="infi-scroll-comp-root">
    <div class="scroll-container" ref="scrollContainer" v-if="!initialLoad">
      <div class="list-container" ref="listContainer">
        <div class="list-item" v-for="(item, index) in list" :key="index">
          {{ item.name }}
        </div>
      </div>
      <div class="sentinel" ref="sentinel"></div>

      <div v-if="canLoadMore" class="loadingMore">LOADING .....</div>
    </div>
    <div class="initialLoad" v-else>INITIAL LIST LOADING .....</div>
  </div>
</template>

<script>
export default {
  created() {
    //load first page
    this.fetchItemsAPI(0, this.pageCount).then((items) => {
      this.list.push(...items);
      this.pageNumber++;
      this.initialLoad = false;
      //wait for initial list to render and then set up observer
      this.$nextTick().then(() => {
        this.setUpInterSectionObserver();
      });
    });
  },
  destroyed() {
    if (this.listEndObserver) {
      this.listEndObserver.disconnect();
    }
  },
  mounted() {},
  data() {
    return {
      list: [],
      /**
       * if we extracted infinite-scroll to a generic component,
       * it would need just these props.loadingMore,canLoadMore and the list
       */

      isLoadingMore: false,
      canLoadMore: true, //list has to end at some point

      //extra stuff
      initialLoad: true, //initial load state ,if you want  to use a different BIGGER loading animation  etc
      pageNumber: 0, //canLoadMore is set to false when last page is reached
      pageCount: 10,
    };
  },
  methods: {
    setUpInterSectionObserver() {
      let options = {
        root: this.$refs["scrollContainer"],
        margin: "10px",
      };
      this.listEndObserver = new IntersectionObserver(
        this.handleIntersection,
        options
      );

      this.listEndObserver.observe(this.$refs["sentinel"]);
    },
    handleIntersection([entry]) {
      if (entry.isIntersecting) {
        console.log("sentinel intersecting");
      }
      if (entry.isIntersecting && this.canLoadMore && !this.isLoadingMore) {
        this.loadMore();
      }
    },
    async loadMore() {
      try {
        this.isLoadingMore = true;
        let items = await this.fetchItemsAPI(this.pageNumber, this.pageCount);
        console.log("loaded page " + this.pageNumber);
        this.pageNumber++;
        this.list.push(...items);
      } catch (error) {
        console.log("Reached end of page", error);
        this.canLoadMore = false;
      } finally {
        this.isLoadingMore = false;
      }
    },

    fetchItemsAPI(pageNumber, pageCount) {
      //assume 10 items per page

      return new Promise((res, rej) => {
        let data = [];
        for (let i = pageNumber * 10; i < pageNumber * 10 + 10; i++) {
          data.push({
            name: "Item " + i,
          });
        }

        setTimeout(() => {
          if (pageNumber < pageCount) {
            res(data);
          } else {
            rej(new Error("No more items to load"));
          }
        }, 1000);
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.scroll-container {
  height: 100%;
  overflow-y: scroll;
}
.sentinel {
  height: 0px;
}

.list-container {
  border: 1px solid darkgray;
}
.list-item {
  margin-bottom: 5px;
  border: 1px solid darkgoldenrod;
  background: lightgoldenrodyellow;
  color: black;
  padding: 15px;
  line-height: 25px;
  text-align: center;
  vertical-align: middle;
  font-size: 18px;
  border-radius: 5px;
}
.loadingMore {
  text-align: center;
}
</style>