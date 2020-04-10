<template>
  <div class="app-container">123</div>
</template>

<script>
export default {
  name: "Database",
  data() {
    return {
      databasePid: ""
    };
  },
  watch: {},
  created() {
    if (location.hash.split("/")[2] === "poorSealingDatabase") {
      // 封孔不良
      this.databasePid = "2";
    } else if (location.hash.split("/")[2] === "goafDatabase") {
      // 矿井及周边采空区
      this.databasePid = "3";
    }
    this.getPage(this.databasePid, "");
  },

  methods: {
    getPage(id, name) {
      console.log(id);
    }
  }
};
</script>

