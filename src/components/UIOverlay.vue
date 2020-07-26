<template>
  <div class="UIOverlay">
    <div class="top-right">
      <div class="button" id="dashboardButton" @click="showDash=true">Dashboard</div>
    </div>
    <div class="bottom-left">
      <div class="category" v-for="cat in categories" :key="cat.category">
        <div class="circle" :style="{ backgroundColor: cat.color }"></div>
        <div class="category-title">{{ cat.category }}</div>
      </div>
    </div>

    <div class='top-centre'>
      <modal v-if="showDash" id="dashModal" class="modal">
        <div class="modalContent">
        <span class="close" @click="showDash = false">&times;</span> 
        <h2> Your newsworthy.ml Dashboard </h2>

        </div>
      </modal>
    </div> 
  </div>
</template>

<script>
export default {
  name: "UIOverlay",
  data() {
    return {
      categories: [
        { category: "COVID-19", color: "#f34c46" },
        { category: "Politics", color: "#fa8d4f" },
        { category: "Business", color: "#fdd742" },
        { category: "Sports", color: "#a3e048" },
        { category: "Arts & Media", color: "#49da9a" },
        { category: "Science & Tech", color: "#50d4fe" },
        { category: "Lifestyle", color: "#6073fd" },
        { category: "Community", color: "#ff95d5" },
        { category: "Crisis Updates", color: "#000000" },
      ],
    };
  },
};
</script>
s

<style scoped>
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
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.4);
}
.modal-content {
  background-color: #fefefe;
  margin: 15% auto;
  padding: 20px;
  border: 1px solid #888;
  width: 80%;
  background-color: #242424;
  color: rgb(228, 228, 228);
  border-radius: 8px;
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
