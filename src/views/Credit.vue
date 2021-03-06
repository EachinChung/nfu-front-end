<template>
  <div>
    <van-nav-bar
      title="已选学分"
      :border="false"
      left-arrow
      @click-left="$router.push('/credit-pro')"
    />
    <div v-if="$store.state.achievement">
      <van-tabs
        v-model="active"
        sticky
        swipeable
        animated
      >
        <van-tab title="专业课">
          <div :style="{height:'20px'}"></div>
          <van-pull-refresh
            v-model="isLoading"
            @refresh="onRefresh"
            :success-text="successText"
          >
            <div class="card">
              <van-cell
                v-if="classification.professionalCore.length!==0"
                title="专业核心课"
                :value="`${total(classification.professionalCore)}`"
                class="cardItem"
              />
              <van-cell
                v-if="classification.professionalDirection.length!==0"
                title="专业方向课"
                :value="`${total(classification.professionalDirection)}`"
                class="cardItem"
              />
              <van-cell
                v-if="classification.professionalElective.length!==0"
                title="专业选修课"
                :value="`${total(classification.professionalElective)}`"
                class="cardItem"
              />
            </div>

            <div class="card" v-if="classification.professionalCore.length!==0">
              <van-cell
                v-for="(item,index) in classification.professionalCore"
                :key="index"
                :title="item.courseName"
                :value="item.credit"
                :label="item.subdivisionType"
                class="cardItem"
              />
            </div>

            <div
              v-if="classification.professionalDirection.length!==0"
              class="card"
            >
              <van-cell
                v-for="(item,index) in classification.professionalDirection"
                :key="index"
                :title="item.courseName"
                :value="item.credit"
                :label="item.subdivisionType"
                class="cardItem"
              />
            </div>

            <div
              v-if="classification.professionalElective.length!==0"
              class="card"
            >
              <van-cell
                v-for="(item,index) in classification.professionalElective"
                :key="index"
                :title="item.courseName"
                :value="item.credit"
                class="cardItem"
                :label="item.subdivisionType"
              />
            </div>
          </van-pull-refresh>
        </van-tab>

        <van-tab title="公共课">
          <div :style="{height:'20px'}"></div>
          <van-pull-refresh
            v-model="isLoading"
            @refresh="onRefresh"
            :success-text="successText"
          >
            <div class="card">
              <van-cell
                v-if="classification.publicCompulsory.length!==0"
                title="公共必修课"
                :value="`${total(classification.publicCompulsory)}`"
                class="cardItem"
              />
              <van-cell
                v-if="classification.growingCompulsory.length!==0"
                title="成长必修课"
                :value="`${total(classification.growingCompulsory)}`"
                class="cardItem"
              />
              <van-cell
                v-if="classification.publicElective.length!==0"
                title="公共选修课"
                :value="`${total(classification.publicElective)}`"
                class="cardItem"
              />
            </div>

            <div
              v-if="classification.publicCompulsory.length!==0"
              class="card"
            >
              <van-cell
                v-for="(item,index) in classification.publicCompulsory"
                :key="index"
                :title="item.courseName"
                :value="item.credit"
                :label="item.subdivisionType"
                class="cardItem"
              />
            </div>

            <div
              v-if="classification.growingCompulsory.length!==0"
              class="card"
            >
              <van-cell
                v-for="(item,index) in classification.growingCompulsory"
                :key="index"
                :title="item.courseName"
                :value="item.credit"
                :label="item.subdivisionType"
                class="cardItem"
              />
            </div>

            <div
              v-if="classification.publicElective.length!==0"
              class="card"
            >
              <van-cell
                v-for="(item,index) in classification.publicElective"
                :key="index"
                :title="item.courseName"
                :value="item.credit"
                :label="item.subdivisionType"
                class="cardItem"
              />
            </div>
          </van-pull-refresh>
        </van-tab>
      </van-tabs>
    </div>
  </div>
</template>

<script>
  import {checkLogin} from "@/network/token";
  import {getAchievement, updateAchievement} from "@/network/achievement";
  import {PullRefresh, Tab, Tabs} from "vant";


  function initAchievement(vm) {
    let classification = {
      professionalElective: [], // 专业选修
      professionalDirection: [],  // 专业方向
      professionalCore: [], // 专业核心
      publicElective: [], // 公共选修
      publicCompulsory: [], // 公共必修
      growingCompulsory: []  // 成长必修
    }
    for (const item of vm.$store.state.achievement) {

      // 去除挂科的课程
      if (item.totalAchievements < 60) {
        if (item.resitExamAchievementPoint == null) continue
        else if (item.resitExamAchievementPoint < 60) continue
      }

      if (item.subdivisionType === '专业选修课') classification.professionalElective.push(item)
      else if (item.subdivisionType === '专业方向课') classification.professionalDirection.push(item)
      else if (item.subdivisionType === '专业核心课') classification.professionalCore.push(item)
      else if (item.courseType === '公共必修') classification.publicCompulsory.push(item)
      else if (item.courseType === '公共选修') classification.publicElective.push(item)
      else if (item.courseType === '成长必修') classification.growingCompulsory.push(item)

    }
    return classification
  }

  export default {
    components: {
      [Tab.name]: Tab,
      [Tabs.name]: Tabs,
      [PullRefresh.name]: PullRefresh
    },
    beforeRouteEnter(to, from, next) {
      checkLogin(to, next)
    },
    data() {
      return {
        active: 0,
        isLoading: false,
        successText: null
      }
    },
    computed: {
      classification() {
        return initAchievement(this)
      }
    },
    mounted() {
      if (this.$store.state.achievement == null) {
        this.$toast.loading({forbidClick: true, duration: 0})
        getAchievement()
          .then(res => {
            if (res.data.code === "1000") this.$store.commit('setAchievement', res.data.message)
            else this.$notify(res.data.message)
            this.$toast.clear()
          })
          .catch(() => {
            this.$notify("无法连接到服务器")
            this.$toast.clear()
          })
      }
    },
    methods: {
      onRefresh() {
        updateAchievement()
          .then(res => {
            if (res.data.code === "1000") {
              this.$store.commit('setAchievement', res.data.message)
              this.successText = "刷新成功"
            } else this.successText = res.data.message
            this.isLoading = false
          })
          .catch(() => {
            this.successText = "无法连接到服务器"
            this.isLoading = false
          })
      }
    }
  }
</script>

<style scoped>
  @import "~@/assets/css/card.css";
</style>