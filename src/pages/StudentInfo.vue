<template>
  <el-row type="flex" width="100px">
    <el-form label-width="80px">
      <div class="FormStuInfoFields">
        <el-form-item label="证件号">
        <el-tag>{{info.sID}}</el-tag>
      </el-form-item>
      <el-form-item label="姓名">
        <el-tag>{{info.sName}}</el-tag>
      </el-form-item>
      <el-form-item label="性别">
        <el-tag>{{info.sex}}</el-tag>
      </el-form-item>
      <el-form-item label="班级">
        <el-tag>{{info.classno}}</el-tag>
      </el-form-item>
      </div>
    </el-form>
  </el-row>
</template>

<script>
export default {
  name: "StudentInfo",

  data() {
    return {
      info: {
        classno: "加载中",
        sID: "加载中",
        sName: "加载中",
        sex: "加载中"
      }
    };
  },

  name: "StudentScore",

  created: function() {
    this.getInfo();
  },

  methods: {
    getInfo() {
      this.$http
        .get("/stu?sID=" + this.$store.state.user.username)
        .then(res => {
          this.info = res.data;
        });
    }
  }
};
</script>

<style>
.FormStuInfoFields {
  width: 300px;
  border-radius: 10px;
  border: 1px solid lightgray;
  background-color: white;
  display: flex;
  flex-direction: column;
  padding: 24px 15px;
}
</style>
