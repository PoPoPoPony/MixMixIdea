<template>
  <div class="container">
    <div class="cl-header">
      <div class="ch-LR">
        <div class="LR-top">
          <span class="before-text">現在時間</span>
          <span class="before-text">5/24 18:00</span>
        </div>

        <div class="LR-bottom">
          <button class="top-button setting"></button>
        </div>
      </div>
      <div class="ch-center">
        <div class="infro-img f-c animated flipInX">
            <img class="circle" src="/img/test_img.jpg"/>
        </div>
        <div class="infro-name animated fadeIn">{{username}}</div>
      </div>
      <div class="ch-LR">
        <div class="LR-top">
          <span class="before-text">上次登入</span>
          <span class="before-text">5/24 18:00</span>
        </div>
        <div class="LR-bottom">
          <button class="top-button shop"></button>          
        </div>
      </div>
    </div>
    
    <div class="cl-body">
      <div class="choose-text f-c">請選擇課程</div>
      <div class="courselist">
      <!-- 這裡開始會用到v-for -->
        <div v-for="(c,index) in courses" class="course" :class="{ open: c.isOpen }">
          <div class="class-infro animated fadeInUp" @click="clickCourseItem(index)">
            <div class="subject f-c"> {{c.department}} </div>
            <div class="class-name f-c"> {{c.name}} </div>
            <div class="class-time f-c">
              <img  class="logo-time" src="/img/time.png" />
              <span class="time-text"> {{c.class_time}} </span>
            </div>
          </div>
          <div class="class-detail animated fadeInDown" :class="{hide : !c.isOpen}">
            <div v-if="loginAs=='teacher'" @click="swapToTW(c.id)" class="col-d">
              <span class="add-room"></span>
              <span class="room-text f-c">創立房間</span>
              </div>
            <div v-if="loginAs=='student'" @click="swapToSW(c.id)" class="col-d">
              <span class="join-room"></span>
              <span class="room-text f-c">加入房間</span>
            </div>
            <div class="col-d">
              <span class="history"></span>
              <span class="room-text f-c">歷史紀錄</span>
            </div>
          </div>
        </div>
      <!-- 這裡開始會用到v-for -->

      </div>

    </div>
  </div >
</template>

<script>
export default {
  data() {
    return {
      username: sessionStorage.getItem("username"),
      courses: [],
      loginAs: sessionStorage.getItem("loginAs")
    };
  },
  methods: {
    clickCourseItem: function(item) {
      this.$set(this.courses[item], "isOpen", !this.courses[item].isOpen);
    },

    swapToTW: function(id) {
      // if no room for this course then create and enter else just enter
      if (sessionStorage.getItem("room_course_id") == null) {
        let self = this;
        self.$nextTick(function() {
          axios
            .post("/courselist/createroom", { course_id: id })
            .then(function(rtn) {
              if (!rtn.data.errmsg) {
                sessionStorage.setItem("room_course_id", id);
                self.$router.push({ path: "/t_waiting" });
              } else {
                console.log(rtn.data.errmsg);
              }
            })
            .catch(function(err) {
              console.log(err);
            });
        });
      }
    },

    swapToSW: function(id) {
      let self = this;
      self.$nextTick(function() {
        axios
          .post("/courselist/findroom", { course_id: id })
          .then(function(rtn) {
            if (!rtn.data.errmsg) {
              console.log(rtn.data.result);
              sessionStorage.setItem("room_course_id", id);
              self.$router.push({ path: "/s_waiting" });
            } else {
              console.log(rtn.data.errmsg);
            }
          })
          .catch(function(err) {
            console.log(err);
          });
      });
    }
  },

  beforeCreate: function() {
    let isLogin = sessionStorage.getItem("loginAs");
    if (!isLogin) this.$router.push({ path: "/" });
  },

  mounted: function() {
    let self = this;
    self.$nextTick(function() {
      axios
        .post("/courselist/getCourses")
        .then(function(rtn) {
          if (!rtn.data.errmsg) {
            self.courses = rtn.data.result;
          } else {
            console.log(rtn.data.errmsg);
          }
        })
        .catch(function(err) {
          console.log(err);
        });
    });
  },
  updated: function() {
    // Andy Test For Update
    console.log("view updated");
  }
};
</script>


<style scoped>
.cl-header {
  width: 100%;
  height: 30%;
}

.cl-body {
  width: 100%;
  height: 70%;
}

.ch-LR {
  float: left;
  width: 26%;
  height: 100%;
}

.ch-center {
  float: left;
  width: 44%;
  height: 100%;
}

.infro-img {
  width: 100%;
  height: 80%;
}

.infro-name {
  color: #34314c;
  font-weight: bold;
  text-align: center;
  font-size: 4em;
  width: 100%;
  height: 20%;
  text-shadow: 0px 4px 3px rgba(0, 0, 0, 0.4), 0px 8px 13px rgba(0, 0, 0, 0.1),
    0px 18px 23px rgba(0, 0, 0, 0.1);
}

.circle {
  width: 18.5em;
  height: 18.5em;
  border-radius: 99em;
  box-shadow: 0px 4px 3px rgba(0, 0, 0, 0.3), 0px 8px 13px rgba(0, 0, 0, 0.1),
    0px 18px 23px rgba(0, 0, 0, 0.1);
}

.LR-top {
  box-sizing: border-box;
  padding: 3%;
  padding-top: 30%;
  width: 100%;
  height: 50%;
}

.LR-bottom {
  width: 100%;
  height: 50%;
}

.before-text {
  width: 100%;
  font-size: 2.5em;
  float: left;
  text-align: center;
  margin: 3%;
  text-shadow: 2px 2px 1px #ccc;
}

.top-button {
  width: 100%;
  height: 100%;
}

.setting {
  background: transparent url(/img/setting.png) no-repeat top center;
  background-size: 50%;
  position: relative;
  left: 20%;
  top: 5%;
}

.shop {
  background: transparent url(/img/shop.png) no-repeat top center;
  background-size: 52%;
  position: relative;
  left: -10%;
  top: 5%;
}

.choose-text {
  font-weight: bold;
  width: 100%;
  height: 12%;
  color: #ff7473;
  font-size: 4.5em;
}

.courselist {
  box-sizing: border-box;
  padding: 8%;
  padding-top: 4%;
  width: 100%;
  height: 88%;
  overflow-y: scroll;
}

.course {
  width: 100%;
  height: 17%;
  box-sizing: border-box;
  padding: 1%;
  margin-bottom: 5%;
  transition: margin-bottom 1.5s;
}

.course.open {
  margin-bottom: 20%;
}

.class-infro {
  position: relative;
  z-index: 2;
  box-sizing: border-box;
  padding: 3%;
  width: 100%;
  height: 100%;
  background-color: #ffc952;
  border-radius: 40px;
  box-shadow: 3px 3px 3px 2px #ccc;
}

.subject {
  font-size: 2.7em;
  float: left;
  width: 25%;
  height: 100%;
}

.class-name {
  font-weight: bold;
  font-size: 3em;
  float: left;
  width: 60%;
  height: 100%;
}

.class-time {
  font-size: 2.3em;
  float: left;
  width: 15%;
  height: 100%;
}

.logo-time {
  width: 40%;
  margin: 3%;
}

.time-text {
  text-align: center;
  line-height: 130%;
}

.class-detail {
  box-sizing: border-box;
  padding: 2%;
  position: sticky;
  z-index: 1;
  margin-top: -2%;
  width: 100%;
  height: 115%;
  background-color: #fbffb9;
  border-radius: 10px;
}

.col-d {
  float: left;
  width: 50%;
  height: 100%;
  font-size: 2.5em;
}

.add-room {
  float: left;
  width: 40%;
  height: 100%;
  background: transparent url("/img/addroom.png") no-repeat center 70%;
  background-size: 45%;
}

.join-room {
  float: left;
  width: 40%;
  height: 100%;
  background: transparent url("/img/joinroom.png") no-repeat center 75%;
  background-size: 40%;
}

.history {
  float: left;
  width: 40%;
  height: 100%;
  background: transparent url("/img/history.png") no-repeat center 75%;
  background-size: 50%;
}

.room-text {
  float: left;
  width: 50%;
  height: 100%;
  font-size: 1.1em;
  padding-top: 5%;
}

.hide {
  display: none;
}
</style>