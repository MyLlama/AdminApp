<template>
  <div>
    <div class="course-details">
      <img :src="course.overviewfiles ? course.overviewfiles.fileurl+`?token=${token}`: require('@/assets/defaultImage.jpg')" alt="" @error="setDefaultImage">
      <div class="course-info">
        <div class="card-body">
          <p class="overline">{{ getStartdate(course.startdate) }}</p>
          <h2 class="course-title">{{ course.fullname }}</h2>
          <p class="course-timeline">{{ getCourseTimeline(course.startdate, course.enddate) }} {{ getCourseTimeline(course.startdate, course.enddate) > 1 ? 'weeks' : 'week'}}</p>
        </div>
      </div>
    </div>

    <hr>

    <div class="course-info">  
      <div class="info-card">
        <p>No. of people</p>
        <h2>{{ enrolledUsers.length }}</h2>
      </div>
    </div>

    <hr>
    
    <div class="course-participants">
      <div v-for='participant in enrolledUsers' :key='participant?.id' class="participant-card" @click='openUser(course.id, participant?.id)'>
        <img :src="participant?.profileimageurl" alt="">
        <h5>{{ participant?.firstname }} {{ participant?.lastname }}</h5>
        <p>{{ participant?.username }}</p>
      </div>
    </div>

    <hr>

    <div class="activity-tracker">
      <table>
        <tr>
          <th>firstname / lastname</th>
          <th>email</th>
          <th v-for="activity in Object.values(this.statuses)[0]?.status" :key="activity">{{ activity.modname }}</th>
        </tr>
        <tr v-for="activity in Object.values(this.statuses)" :key="activity.email">
          <td>{{ activity.fullname }}</td>
          <td>{{ activity.email }}</td>
          <td v-for='status in activity.status' :key='status.modname'>
            <input type="checkbox" name="" id="" :checked="status.status">
          </td>
        </tr>
        <tr></tr>
      </table>
    </div>
  </div>
</template>
<script>
import moodleService from '../services/moodleService';

export default {
  name: 'CourseDetails',
  data() {
    return {
      course: {},
      token: localStorage.getItem('token'),
      enrolledUsers: [],
      statuses: {}
    };
  },
  async mounted(){
    const id = this.$route.params.id;
    this.course = await this.getCourseData(id);
    this.getCourseTimeline(this.course.startdate, this.course.enddate);
    this.getEnrolledUsers(this.course?.id);
  }, 
  methods: {
    setDeafaultImage(event) {
      event.target.src = require('@/assets/defaultImage.jpg')
    },
    async getActivitiesCompletionStatus(){
     const courseId = this.$route.params.id;
     this.enrolledUsers.map(async (user) => {
        const resp = await moodleService.getActivitiesCompletionStatus(courseId, user.id);
        this.statuses[user.id] = {status: await resp};
        this.statuses[user.id] = {...this.statuses[user.id], fullname: user.fullname, email: user.email };
     })
    },
    openUser(id, userid){
      this.$router.push(`/user/${id}/${userid}`);
    },
    getStartdate(startdate){
      const date = new Date(startdate * 1000);
      var months = ['JANUARY', 'FEBRUARY', 'MARCH', 'APRIL', 'MAY', 'JUNE', 'JULY', 'AUGUST', 'SEPTEMBER', 'OCTOBER', 'NOVEMBER', 'DECEMBER'];
      return `${months[date.getMonth()]} ${date.getDate()}, ${date.getFullYear()}`;
    },
    getCourseTimeline(startDate, endDate){
      startDate = new Date(startDate * 1000).getTime();
      endDate = new Date(endDate * 1000).getTime();

      const days = Math.ceil((endDate - startDate)/(1000*60*60*24));
      const weeks = Math.ceil(days/7);

      return weeks;

    },
    async getEnrolledUsers(id){
      this.enrolledUsers = await moodleService.countEnrolledUsers(id);
      await this.getActivitiesCompletionStatus(this.enrolledUsers);
    },
    async getCourseData(id){
      const courses = await moodleService.getCourseData(this.$route.params.userid);
      return courses.find(course => course.id == id);
    }
  }  
};
</script>
<style scoped>
.course-details {
  margin: 30px;
  display: flex;
  align-items: center;
  line-height: 30px;
}

.course-details img {
  height: 150px;
  width: 150px;
  border-radius: 10px;
  margin-right: 20px;
}

.overline {
  font-size: 14px;
  font-weight: 600;
  color: #F47431;
}

.course-timeline {
  font-size: 14px;
  color: grey;
}

hr {
  margin: 30px;
  height: 2px;
  border: none;
  background-color: #d8dadb
}

.course-info {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
}

.info-card {
  width: 280px;
  border: 1px solid grey;
  border-radius: 10px;
  padding: 20px;
  margin: 0px 30px;
  background-color: #FFF;
}

.info-card p{
  font-size: 16px;
  font-weight: 500;
}

.course-participants {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  text-align: center;
  margin: 0px 30px;
}
.participant-card {
  margin-bottom: 20px;
  padding: 20px 0px;
  border-radius: 10px;
}

.participant-card img {
  border-radius: 100%;
}

.participant-card:hover {
  background-color: #d8dadb;
  cursor: pointer;
}

.activity-tracker table {
  width: 90%;
}

table td, table th {
  padding: 10px;
  text-align: center;
  pointer-events: none;
}
</style>
  