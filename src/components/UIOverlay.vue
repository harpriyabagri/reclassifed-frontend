<template>
  <div class="UIOverlay">
    <div class="top-right">
      <div class="button" id="aboutButton" @click="showAbout = true">About</div>
      <div class="button">Dashboard</div>
    </div>
    <div class="bottom-left">
      <div class="logo">
        <img src="@/assets/Newsworthy.ml.png" />
      </div>
      <div
        class="category"
        v-for="cat in categories"
        :key="cat.category"
        @click="filter(cat)"
        :style="[cat.include ? {backgroundColor: cat.bgColor} : {backgroundColor: null}]"
      >
        <div class="circle" :style="{ backgroundColor: cat.color }"></div>
        <div class="category-title">{{ cat.category }}</div>
      </div>
    </div>
    <div class="top-centre">
      <modal v-if="showAbout" id="aboutModal" class="modal">
        <div class="modal-content">
          <span class="close" @click="showAbout = false">&times;</span>
          <h2>about newsworthy.ml</h2>
          <p>
            Hey, thanks for checking out Newsworthy! This project is the brainchild of six fellows working at
            the Montreal-based AI4Good Lab. Following an intensive five-week curriculum of AI and machine learning,
            we had three weeks to implement Newsworthy as a working product. We hope you enjoy exploring Newsworthy
            as much as we enjoyed creating it!
            <br />
            <br />Now more than ever before, information literacy is crucial. Newsworthy provides analytics on users
            reading habits and provides suggestions on how users can diversify their news intake to reduce bias, and
            improve their information literacy. Did you know that nearly half of original news stories are first broken by
            local news outlets? Despite this, local newspapers are being eradicated by larger national outlets. Newsworthy
            strives to provide an equal playing field for local newspapers, so they can get the attention they deserve.
            We had three weeks to create Newsworthy, and think that if we had four, five, gosh forbid six weeks, we could
            do a lot more.
            <br />
            <br />Future implementations and ideas include _____ and ______.
            <br />
            <br />Nobody is perfect, and we are no exception! We have identified several limitations of our project.
            (we can list these as a group and I can add it in.)
          </p>
        </div>
      </modal>
      <div class="search">
        <form action>
          <input type="text" placeholder="Search..." name="search" />
          <!-- <button type="submit"></button> -->
        </form>
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
        {
          category: "COVID-19",
          color: "#f34c46",
          index: 0,
          include: true,
          realCategory: "covid-19",
          bgColor: 'rgba(243,76,70,0.6)'},
        {
          category: "Politics",
          color: "#fa8d4f",
          index: 1,
          include: true,
          realCategory: "politics",
          bgColor: 'rgba(251,141,79,0.6)' },
        {
          category: "Business",
          color: "#fdd742",
          index: 2,
          include: true,
          realCategory: "business",
          bgColor: 'rgba(253,215,66,0.5)' },
        {
          category: "Sports",
          color: "#a3e048",
          index: 3,
          include: true,
          realCategory: "sports",
          bgColor: 'rgba(163,224,72,0.6)' },
        {
          category: "Arts & Media",
          color: "#2EC582",
          index: 4,
          include: true,
          realCategory: "arts & entertainment",
          bgColor: 'rgba(46,197,130,0.5)' },
        {
          category: "Science & Tech",
          color: "#50d4fe",
          index: 5,
          include: true,
          realCategory: "science",
          bgColor: 'rgba(80,212,254,0.6' },
        {
          category: "Lifestyle",
          color: "#6073fd",
          index: 6,
          include: true,
          realCategory: "lifestyle",
          bgColor: 'rgba(96,115,253,0.5)' },
        {
          category: "Community",
          color: "#ff95d5",
          index: 7,
          include: true,
          realCategory: "local",
          bgColor: 'rgba(255,149,213,0.5)' },
        {
          category: "Crisis Updates",
          color: "#000000",
          index: 8,
          include: true,
          realCategory: "crisis-updates",
          bgColor: 'rgba(0,0,0,0.4)' },
      ],
      active_filters: [0, 0, 0, 0, 0, 0, 0, 0, 0],
      showAbout: false,
    };
  },
  methods: {
    filter(cat) {
      for (let i in this.categories) {
        if (cat.category === this.categories[i].category) {
          this.categories[i].include = !this.categories[i].include;
        }
      }
      this.$emit("revaluate-filters", this.categories);
    },
  },
  mounted() {
    this.$emit("revaluate-filters", this.categories);
  },
};
</script>


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
.top-centre {
  pointer-events: auto;
  position: absolute;
  left: 40%;
  top: 10px;
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
  width: 100%;
}
.category:hover {
  background-color: #f2f2f2;
}
.category-title {
  opacity: 1.0;
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
input[type="text"] {
  background: rgba(0, 0, 0, 0);
  background-color: none;
  color: #414141;
  height: 30px;
  width: 200px;
  letter-spacing: 0.4px;
  /* border-radius: 5px; */
  font-size: 16px;
  letter-spacing: 0.4px;
  outline: none;
  box-sizing: border-box;
  font-family: "Times New Roman";
  justify-content: flex-start;
  padding: 0 5px;
  border: 0;
  outline: 0;
  border-bottom: 1px solid #242424;
  margin: auto;
}
::placeholder {
  color: #b0b0b0;
  font-family: "Times New Roman";
}
img {
  width: 180px;
  height: auto;
}
</style>
