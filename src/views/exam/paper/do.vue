<template>
  <div class="my-do-exam">
      <div class="my-do-exam-info">
        <h3>{{form.name}}</h3>
        <div class="my-do-exam-info-sore">
          <span style="margin-right: 18px">总分：{{form.score}}</span>
          <span>时长：{{form.suggestTime}}分钟</span>
        </div>
        <el-divider></el-divider>
        <span :key="item.itemOrder"  v-for="item in answer.answerItems">
             <el-tag :type="questionCompleted(item.completed)" class="do-exam-title-tag" @click="goAnchor('#question-'+item.itemOrder)">{{item.itemOrder}}</el-tag>
        </span>
        <el-divider></el-divider>
        <div class="my-do-exam-info-time">
          <label>剩余时间</label><br>
          <label style="font-size: 18px; color: red;">{{formatSeconds(remainTime)}}</label>
        </div>
        <el-divider></el-divider>
        <el-row class="do-align-center">
             <el-button type="primary" @click="submitForm">提交试卷</el-button>
           </el-row>
      </div>
      <!-- <el-col :span="8">
        <span :key="item.itemOrder"  v-for="item in answer.answerItems">
             <el-tag  class="do-exam-title-tag" >{{item.itemOrder}}</el-tag>
        </span>
        <span class="do-exam-time">
          <label>剩余时间：</label>
        </span>
      </el-col> -->
    <el-container class="my-do-exam-content">
      <el-main>
        <el-form :model="form" ref="form" v-loading="formLoading" label-width="100px">
          <el-row :key="index"  v-for="(titleItem,index) in form.titleItems">
            <p class="my-do-exam-content-title">{{titleItem.name}}</p>
            <div class="exampaper-item-box" v-if="titleItem.questionItems.length!==0">
              <el-form-item :key="questionItem.itemOrder" :label="questionItem.itemOrder+'.'"
                            v-for="questionItem in titleItem.questionItems"
                            class="exam-question-item" label-width="50px" :id="'question-'+ questionItem.itemOrder">
                <QuestionEdit :qType="questionItem.questionType" :question="questionItem"
                              :answer="answer.answerItems[questionItem.itemOrder-1]"/>
                <el-divider class="hr"></el-divider>
              </el-form-item>
            </div>
          </el-row>
           <!-- <el-row class="do-align-center">
             <el-button type="primary" @click="submitForm">提交答卷</el-button>
           </el-row> -->
        </el-form>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import { mapState, mapGetters } from 'vuex'
import { formatSeconds } from '@/utils'
import QuestionEdit from '../components/QuestionEdit'
import examPaperApi from '@/api/examPaper'
import examPaperAnswerApi from '@/api/examPaperAnswer'

export default {
  components: { QuestionEdit },
  data () {
    return {
      form: {},
      formLoading: false,
      answer: {
        questionId: null,
        doTime: 0,
        answerItems: []
      },
      timer: null,
      remainTime: 0
    }
  },
  created () {
    let id = this.$route.query.id
    let _this = this
    if (id && parseInt(id) !== 0) {
      _this.formLoading = true
      examPaperApi.select(id).then(re => {
        _this.form = re.response
        _this.remainTime = re.response.suggestTime * 60
        _this.initAnswer()
        _this.timeReduce()
        _this.formLoading = false
      })
    }
  },
  mounted () {

  },
  beforeDestroy () {
    window.clearInterval(this.timer)
  },
  methods: {
    formatSeconds (theTime) {
      return formatSeconds(theTime)
    },
    timeReduce () {
      let _this = this
      this.timer = setInterval(function () {
        if (_this.remainTime <= 0) {
          _this.submitForm()
        } else {
          ++_this.answer.doTime
          --_this.remainTime
        }
      }, 1000)
    },
    questionCompleted (completed) {
      return this.enumFormat(this.doCompletedTag, completed)
    },
    goAnchor (selector) {
      this.$el.querySelector(selector).scrollIntoView({ behavior: 'instant', block: 'center', inline: 'nearest' })
    },
    initAnswer () {
      this.answer.id = this.form.id
      let titleItemArray = this.form.titleItems
      for (let tIndex in titleItemArray) {
        let questionArray = titleItemArray[tIndex].questionItems
        for (let qIndex in questionArray) {
          let question = questionArray[qIndex]
          this.answer.answerItems.push({ questionId: question.id, content: null, contentArray: [], completed: false, itemOrder: question.itemOrder })
        }
      }
    },
    submitForm () {
      let _this = this
      window.clearInterval(_this.timer)
      _this.formLoading = true
      examPaperAnswerApi.answerSubmit(this.answer).then(re => {
        if (re.code === 1) {
          _this.$alert('试卷得分：' + re.response + '分', '考试结果', {
            confirmButtonText: '返回考试记录',
            callback: action => {
              _this.$router.push('/record/index')
            }
          })
        } else {
          _this.$message.error(re.message)
        }
        _this.formLoading = false
      }).catch(e => {
        _this.formLoading = false
      })
    }
  },
  computed: {
    ...mapGetters('enumItem', ['enumFormat']),
    ...mapState('enumItem', {
      doCompletedTag: state => state.exam.question.answer.doCompletedTag
    })
  }
}
</script>

<style lang="scss" scoped>
  .align-center {
    text-align: center
  }

  .exam-question-item {
    padding: 10px;

    .el-form-item__label {
      font-size: 15px !important;
    }
  }

  .question-title-padding {
    padding-left: 25px;
    padding-right: 25px;
  }

  .my-do-exam {
    box-sizing: border-box !important;
    background-color: rgb(239, 243, 247);
    padding: 24px 80px;
    position: relative;
    &-info {
      position: fixed;
      background-color: white;
      width: 268px;
      height: 672px;
      border: 1px solid #ebeef5;
      border-radius: 4px;
      padding: 20px;
      &-sore {
        font-size: 14px;
      }
      &-time {
        font-size: 14px;
        display: flex;
        flex-direction: column;
        align-items: center;
      }
    }
    &-content {
      // height: 662px;
      margin-left: 300px;
      border: 1px solid #ebeef5;
      border-radius: 4px;
      background-color: white;
      &-title {
        background: #fafafa;
        width:100%;
        height:60px;
        padding-left: 30px;
        line-height:60px;
        font-size:18px;
        border-top: 1px solid #dedede;
        border-bottom: 1px solid #dedede;
      }
    }
  }
  .hr {
    margin-left: -28px;
    margin-bottom: 0px;
  }
</style>
