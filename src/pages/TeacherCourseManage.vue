<template>
  <el-container direction='vertical'>
    <el-button-group>
      <el-dropdown @command="chooseCourse">
        <el-button type="success">
          {{curCourse.cName}}
          <i class="el-icon-arrow-down el-icon--right"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item v-for="(item, index) in courseInfo" :key="item.id" :command="index">{{item.cName}}
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
      <el-button type='primary' icon='el-icon-circle-plus' @click='addScore()'>添加成绩</el-button>
    </el-button-group>

    <el-dialog :title='teaFormTitle' :visible.sync='teaFormVisible'>
      <el-form ref='teaForm' :rules='teaFormRules' :model='teaFormModel' label-width="80px">
        <el-form-item prop='sID' label='学生号'>
          <el-input :disabled="isEdit" v-model='teaFormModel.sID'></el-input>
        </el-form-item>
        <el-form-item prop='grade' label='成绩'>
          <el-input v-model='teaFormModel.score'></el-input>
        </el-form-item>
      </el-form>
      <div slot='footer'>
        <el-button @click='hideStuForm()'>取消</el-button>
        <el-button v-if='isAdd' type='primary' @click='commitAdd()'>确定添加</el-button>
        <el-button v-else type='primary' @click='commitEdit()'>确定编辑</el-button>
      </div>
    </el-dialog>

    <el-table :data='score' stripe max-height='500'>
      <el-table-column sortable prop='sID' label='学号' width='90' />
      <el-table-column prop='sName' label='姓名' width='70' />
      <el-table-column prop='classno' label='班级' width='150' />
      <el-table-column prop='grade' label='成绩' />
      <el-table-column fixed='right' label='操作' width="200">
        <template slot-scope="scope">
          <el-button icon="el-icon-edit" size="mini" @click="editScore(scope.$index, scope.row)">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="ChartFields">
      <h1>条状图</h1>
      <bar-chart ref="barchart" :chart-data="barChartData" :options="{responsive: true, maintainAspectRatio: false}"/>

      <h1>饼状图</h1>
      <pie-chart ref="piechart" :chart-data="pieChartData" :options="{responsive: true, maintainAspectRatio: false}"/>
    </div>

  </el-container>
</template>


<script>
  import BarChart from "@/components/BarChart.js";
  import PieChart from "@/components/PieChart.js";

  export default {
    components: {
      BarChart,
      PieChart
    },
    name: "TeacherScore",

    data() {
      return {
        barChartData: {
          labels: ["0-60分", "60-70分", "70-80分", "80-90分", "90-100分"],
          datasets: [{
            label: "学生成绩分布",
            backgroundColor: "#f87979",
            data: []
          }]
        },
        pieChartData: {
          labels: ["0-60分", "60-70分", "70-80分", "80-90分", "90-100分"],
          datasets: [{
            backgroundColor: [
              "#969696",
              "#41B883",
              "#E46651",
              "#00D8FF",
              "#DD1B16"
            ],
            data: []
          }]
        },
        score: [{
          sID: "",
          sName: "",
          classno: "",
          grade: ""
        }],
        courseInfo: [{
          cName: "",
          cID: ""
        }],
        curCourse: "",
        teaFormVisible: false,
        teaFormTitle: "",
        isAdd: "false",
        isEdit: "false",
        teaFormModel: {
          sID: "",
          sName: "",
          classno: "",
          score: ""
        },
        teaFormRules: {
          sID: [{
            required: true,
            message: "学生号不能为空",
            trigger: "blur"
          }],
          sName: [{
            required: true,
            message: "学生姓名不能为空",
            trigger: "blur"
          }],
          classno: [{
            required: true,
            message: "班级不能为空",
            trigger: "blur"
          }],
          score: [{
            required: true,
            message: "成绩不能为空",
            trigger: "blur"
          }]
        },
        scoreModel: {
          tID: "",
          sID: "",
          cID: "",
          newGrade: ""
        }
      };
    },

    created: function () {
      this.getCourse();
    },

    methods: {
      getCourse: function () {
        this.$http
          .get("/tea/cInfo?tID=" + this.$store.state.user.username)
          .then(res => {
            //console.log(res);
            this.courseInfo = res.data;
            this.curCourse = this.courseInfo[0];
            this.getScore();
          });
      },

      getScore: function () {
        //console.log("getScore cID=" + this.curCourse.cID);
        this.$http
          .get(
            "/tea/getGradeByCID?tID=" +
            this.$store.state.user.username +
            "&cID=" +
            this.curCourse.cID
          )
          .then(res => {
            console.log(res);
            this.score = res.data;
          });
        this.$http
          .get(
            "/tea/pic?tID=" +
            this.$store.state.user.username +
            "&cID=" +
            this.curCourse.cID
          )
          .then(res => {
            this.pieChartData.datasets[0].data = res.data;
            this.barChartData.datasets[0].data = res.data;
            this.$refs.piechart.update();
            this.$refs.barchart.update();
            //console.log(this.pieChartData.datasets.data);
            //console.log(this.barChartData.datasets.data);
          });
      },

      chooseCourse: function (i) {
        console.log("index=" + i);
        this.curCourse = this.courseInfo[i];
        this.getScore();
      },
      hideStuForm: function () {
        this.teaFormVisible = false;
      },
      editScore: function (index, row) {
        this.teaFormTitle = "编辑成绩";
        this.isEdit = true;
        this.isAdd = false;
        this.teaFormVisible = true;
        this.teaFormModel = this.score[index];
      },
      addScore: function (index, row) {
        //console.log(index);
        //console.log(row);
        this.teaFormTitle = "添加成绩";
        this.isEdit = false;
        this.isAdd = true;
        this.teaFormVisible = true;
        this.teaFormModel = {
          sID: "",
          sName: "",
          classno: "",
          score: ""
        };
      },
      commitEdit: function () {
        this.$refs.teaForm.validate(valid => {
          if (valid) {
            console.log("edit");
            this.teaFormVisible = false;

            this.scoreModel.tID = this.$store.state.user.username;
            this.scoreModel.sID = this.teaFormModel.sID;
            this.scoreModel.cID = this.curCourse.cID;
            this.scoreModel.newGrade = this.teaFormModel.score;

            this.$http.post("/tea/editGrade", this.scoreModel).then(res => {
              if (res.data == "成功") {
                this.$message({
                  type: "success",
                  message: "编辑成绩成功",
                  showClose: true,
                  center: true
                });
                this.getScore();
              } else {
                this.$message({
                  type: "error",
                  message: "编辑学生失败",
                  showClose: true,
                  center: true
                });
              }
            });
          } else {
            return false;
          }
        });
      },
      commitAdd: function () {
        this.$refs.teaForm.validate(valid => {
          if (valid) {
            console.log("add");
            this.teaFormVisible = false;

            this.scoreModel.tID = this.$store.state.user.username;
            this.scoreModel.sID = this.teaFormModel.sID;
            this.scoreModel.cID = this.curCourse.cID;
            this.scoreModel.newGrade = this.teaFormModel.score;

            this.$http.put("/tea/newGrade", this.scoreModel).then(res => {
              if (res.data == "成功") {
                this.$message({
                  type: "success",
                  message: "编辑成绩成功",
                  showClose: true,
                  center: true
                });
                this.getScore();
              } else {
                this.$message({
                  type: "error",
                  message: "编辑学生失败",
                  showClose: true,
                  center: true
                });
              }
            });
          } else {
            return false;
          }
        });
      }
    }
  };

</script>

<style>
  .ChartFields {
    border-radius: 10px;
    border: 1px solid lightgray;
    background-color: white;
    display: flex;
    flex-direction: column;
    margin: 10px;
    padding: 30px 15px;
  }

</style>
