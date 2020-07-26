<template>
  <div class="UIOverlay">
    <div class="top-right">
      <div class="button">Dashboard</div>
      <div class="button" id="aboutButton" @click="showAbout">About</div>
    </div>
    <div class="bottom-left">
      <div
        class="category"
        v-for="cat in categories"
        :key="cat.category"
        @click="filter(cat)"
        :class="{ toggled: active_filters[cat.index] }"
      >
        <div class="circle" :style="{ backgroundColor: cat.color }"></div>
        <div class="category-title">{{ cat.category }}</div>
      </div>
    </div>
    <div class="top-centre">
      <div id="aboutModal" class="modal">
        <div class="modal-content">
          <span class="close" @click="close">&times;</span>
          <p>all about our product :)</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "UIOverlay",
  data() {
    return {
      categories: [
        { category: "COVID-19", color: "#f34c46", index: 0 },
        { category: "Politics", color: "#fa8d4f", index: 1 },
        { category: "Business", color: "#fdd742", index: 2 },
        { category: "Sports", color: "#a3e048", index: 3 },
        { category: "Arts & Media", color: "#49da9a", index: 4 },
        { category: "Science & Tech", color: "#50d4fe", index: 5 },
        { category: "Lifestyle", color: "#6073fd", index: 6 },
        { category: "Community", color: "#ff95d5", index: 7 },
        { category: "Crisis Updates", color: "#000000", index: 8 },
      ],
      active_filters: [0, 0, 0, 0, 0, 0, 0, 0, 0],
    };
  },
  methods: {
    filter(cat) {
      var i = cat.index;
      this.$set(this.active_filters, i, !this.active_filters[i])
      this.$emit("revaluate-filters", this.active_filters);
    },
    showAbout() {
      var modal = document.getElementById("aboutModal")
      modal.style.display = "block"
    },
    close() {
      var modal = document.getElementById("aboutModal")
      modal.style.display = "none"
    }
  },
  mounted() {},
};

</script>


<style scoped>
.toggled{
  background-color: white;
}
.UIOverlay {
  position: absolute;
  pointer-events: none;
  width: 100%;
  height: 100%;
}
.top-right {
  pointer-events: auto;
  position: absolute;
  top: 10px;
  right: 50px;
}
.bottom-left {
  pointer-events: auto;
  position: absolute;
  bottom: 50px;
  left: 10px;
}
.top-centre{
  pointer-events: auto;
  position: absolute;
  top: 10px;
  right: 500px;
}
.button {
  letter-spacing: 0.4px;
  background-color: none;
  color: #414141;
  box-shadow: 0 3px 6px #3535353b;
  border-radius: 5px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 6px 12px;
  user-select: none;
  cursor: pointer;
  transition: all;
  transition-duration: 150ms;
}
.button:hover {
  background-color: #f2f2f2;
}

.category {
  letter-spacing: 0.4px;
  background-color: none;
  color: #414141;
  box-shadow: 0 3px 6px #3535353b;
  border-radius: 5px;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  padding: 6px 12px;
  user-select: none;
  cursor: pointer;
  transition: all;
  transition-duration: 150ms;
  margin-bottom: 5px;
}
.category:hover {
  background-color: #f2f2f2;
}
.circle {
  border-radius: 50%;
  width: 15px;
  height: 15px;
  margin-right: 10px;
}
.modal {
  display: none; 
  position: fixed; 
  z-index: 1; 
  left: 0;
  top: 0;
  width: 100%;
  height: 100%; 
  overflow: auto; 
  background-color: rgb(0,0,0); 
  background-color: rgba(0,0,0,0.4); 
}
.modal-content {
  background-color: #fefefe;
  margin: 15% auto; 
  padding: 20px;
  border: 1px solid #888;
  width: 80%; 
}
.close {
  color: #aaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: black;
  text-decoration: none;
  cursor: pointer;
}
</style>
