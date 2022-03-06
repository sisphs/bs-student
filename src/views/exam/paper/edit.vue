<template>
  <div class="my-read-exam">
    <div class="my-read-exam-info">
      <h3>{{form.name}}</h3>
      <div class="my-read-exam-info-sore">
        <span style="margin-bottom: 10px;">试卷得分：{{answer.score}} / {{form.score}}</span>
        <span>试卷耗时：{{formatSeconds(answer.doTime)}}</span>
      </div>
      <el-divider></el-divider>
      <span :key="item.itemOrder"  v-for="item in answer.answerItems">
          <el-tag :type="questionDoRightTag(item.doRight)" class="do-exam-title-tag" @click="goAnchor('#question-'+item.itemOrder)">{{item.itemOrder}}</el-tag>
      </span>
      <el-divider></el-divider>
      <el-row class="do-align-center">
        <el-button type="primary" @click="submitForm">提交试卷</el-button>
      </el-row>
    </div>
    <!-- <el-row  class="do-exam-title-hidden">
      <el-col :span="24">
          <span :key="item.itemOrder"  v-for="item in answer.answerItems">
              <el-tag  class="do-exam-title-tag" >{{item.itemOrder}}</el-tag>
          </span>
      </el-col>
    </el-row> -->
    <el-container  class="my-read-exam-content">
      <el-main>
        <el-form :model="form" ref="form" v-loading="formLoading" label-width="100px">
          <el-row :key="index"  v-for="(titleItem,index) in form.titleItems">
            <p class="my-read-exam-content-title">{{titleItem.name}}</p>
            <div class="exampaper-item-box" v-if="titleItem.questionItems.length!==0">
              <el-form-item :key="questionItem.itemOrder" :label="questionItem.itemOrder+'.'"
                            v-for="questionItem in titleItem.questionItems"
                            class="exam-question-item"  label-width="50px" :id="'question-'+ questionItem.itemOrder">
                <el-row>
                  <QuestionAnswerShow :qType="questionItem.questionType" :question="questionItem"  :answer="answer.answerItems[questionItem.itemOrder-1]"/>
                </el-row>
                <el-row v-if="answer.answerItems[questionItem.itemOrder-1].doRight === null">
                  <label style="color: #e6a23c">批改：</label>
                  <el-radio-group v-model="answer.answerItems[questionItem.itemOrder-1].score">
                    <el-radio  v-for="item in scoreSelect(questionItem.score)"  :key="item"  :label="item" >
                      {{item}}
                    </el-radio>
                  </el-radio-group>
                </el-row>
                <el-divider class="hr"></el-divider>
              </el-form-item>
            </div>
          </el-row>
        </el-form>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import { mapState, mapGetters } from 'vuex'
import { formatSeconds } from '@/utils'
import QuestionAnswerShow from '../components/QuestionAnswerShow'
import examPaperAnswerApi from '@/api/examPaperAnswer'
export default {
  components: { QuestionAnswerShow },
  data () {
    return {
      form: {},
      formLoading: false,
      answer: {
        id: null,
        score: 0,
        doTime: 0,
        answerItems: [],
        doRight: false
      }
    }
  },
  created () {
    let id = this.$route.query.id
    let _this = this
    if (id && parseInt(id) !== 0) {
      _this.formLoading = true
      examPaperAnswerApi.read(id).then(re => {
        _this.form = re.response.paper
        _this.answer = re.response.answer
        _this.formLoading = false
      })
    }
  },
  methods: {
    submitForm () {
      let _this = this
      _this.formLoading = true
      examPaperAnswerApi.edit(this.answer).then(re => {
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
    },
    scoreSelect (score) {
      let array = []
      for (let i = 0; i <= parseInt(score); i++) {
        array.push(i.toString())
      }
      if (score.indexOf('.') !== -1) {
        array.push(score)
      }
      return array
    },
    formatSeconds (theTime) {
      return formatSeconds(theTime)
    },
    questionDoRightTag (status) {
      return this.enumFormat(this.doRightTag, status)
    },
    goAnchor (selector) {
      this.$el.querySelector(selector).scrollIntoView({ behavior: 'instant', block: 'center', inline: 'nearest' })
    }
  },
  computed: {
    ...mapGetters('enumItem', ['enumFormat']),
    ...mapState('enumItem', {
      doRightTag: state => state.exam.question.answer.doRightTag
    })
  }
}
</script>

<style lang="scss" scoped>
  .align-center {
    text-align: center
  }

  .exam-question-item{
    padding: 10px;
    .el-form-item__label{
      font-size: 15px !important;
    }
  }

  .question-title-padding{
    padding-left: 25px;
    padding-right: 25px;
  }
  .my-read-exam {
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
        height: 40px;
        font-size: 14px;
        display: flex;
        flex-direction: column;
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
