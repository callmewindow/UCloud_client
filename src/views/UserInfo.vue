<template>
  <div id="CL">
    <Navigator active-func="user" />

    <el-row :gutter="20" style="margin-left: 15%; margin-right: 15%">
      <el-col style="margin-top: 80px">
        <el-card>
          <el-row>
            <el-col :span="6" :offset="2" style="padding-top: 5px"
              >用户名：{{ username }}</el-col
            >
            <el-col :span="6" style="padding-top: 5px"
              >拥有BOT总数：{{ bot_total }}</el-col
            >
            <el-col :span="6" style="padding-top: 5px"
              >运行中：{{ bot_run }}</el-col
            >
            <el-col :span="4">
              <el-button type="danger" size="small" @click="quit_login"
                >注销</el-button
              >
            </el-col>
          </el-row>
        </el-card>
      </el-col>
    </el-row>

    <el-row style="margin-left: 16%; margin-right: 16%; margin-top: 10px">
      <el-tabs v-model="activeName">
        <el-tab-pane label="我的BOT" name="1"></el-tab-pane>
        <el-tab-pane label="收藏的BOT" name="2">
          功能开发中...
        </el-tab-pane>
      </el-tabs>
    </el-row>

    <el-row
      :gutter="20"
      style="margin-left: 15%; margin-right: 15%; padding-bottom: 50px"
      v-if="activeName === '1'"
    >
      <el-col>
        <el-row :gutter="20">
          <el-col
            :span="8"
            v-for="(bot, index) in bot_list"
            :key="index"
            style="margin-top: 20px"
          >
            <el-card>
              <el-row style="margin: 0">
                <el-col :span="8">
                  <img
                    src="@/assets/ucloud-logo-mini.png"
                    style="float: left"
                  />
                </el-col>
                <el-col :span="16">
                  <el-row style="margin-top: 5px; margin-bottom: 15px">
                    <el-tag
                      type="success"
                      size="small"
                      v-if="bot.botStatus"
                      style="float: left"
                      >运行中</el-tag
                    >
                    <el-tag
                      type="danger"
                      size="small"
                      v-if="!bot.botStatus"
                      style="float: left"
                      >未运行</el-tag
                    >
                    <el-link
                      type="primary"
                      :underline="false"
                      style="font-size: 18px; margin-left: 10px"
                      @click="frontTool.toPath('/bot/' + bot.botId)"
                      >{{ bot.botName }}
                    </el-link>
                  </el-row>
                  <div style="font-size: 15px">
                    <el-row v-if="bot.botType === 1">类型：私聊BOT</el-row>
                    <el-row v-if="bot.botType === 2">类型：群聊BOT</el-row>
                    <el-row v-if="bot.botType === 3">类型：代码检查BOT</el-row>
                    <el-row style="margin-top: 10px">
                      QQ：{{ bot.botQQ }}
                    </el-row>
                  </div>
                </el-col>
              </el-row>
            </el-card>
          </el-col>
        </el-row>
      </el-col>
    </el-row>
    <Footer position="center" />
  </div>
</template>

<script>
import Navigator from "@/components/Navigator";
import Footer from "@/components/Footer";
import * as frontTool from "@/tools/frontTool";
import * as userAPI from "@/APIs/user";

export default {
  name: "UserInfo",
  components: {
    Navigator,
    Footer,
  },
  data() {
    return {
      frontTool,
      username: "123123123",
      bot_total: 0,
      bot_run: 0,
      bot_list: [],
      activeName: "1",
    };
  },
  created() {
    // 填充旧数据
    let oldUI = JSON.parse(localStorage.getItem("userInfo"));
    if (oldUI) {
      this.$store.state.username = oldUI.username;
      this.$store.state.userId = oldUI.userId;
      this.get_user_info();
    } else {
      this.$message("请登录后查看更多内容");
      this.$router.push({ path: "/Login" });
    }
  },
  filters: {
    cut(str) {
      if (str) str = str.slice(0, 10);
      return str;
    },
    cut_8(str) {
      if (str && str.length > 8) str = str.slice(0, 8) + "...";
      return str;
    },
  },
  methods: {
    async get_user_info() {
      try {
        //const res = await userAPI.getUserInfo(1)
        const res = await userAPI.getUserInfo(this.$store.state.userId);
        this.username = this.$store.state.username;
        //window.console.log(res.data);
        if (res.data.success) {
          this.bot_list = res.data.data;
          this.bot_total = this.bot_list.length;
          for (let i = 0; i < this.bot_total; i++)
            if (this.bot_list[i].botStatus) this.bot_run++;
        }
      } catch (e) {
        this.$message.error("请求超时");
      }
    },
    quit_login() {
      this.$confirm("是否确认退出当前账号", "退出登录", {
        confirmButtonText: "确认",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          this.$message.success("已退出登录");
          this.$store.state.userId = -1;
          localStorage.removeItem("userInfo");
          this.$router.push({
            path: "/home",
          });
        })
        .catch(() => {});
    },
  },
};
</script>

<style scoped>
#CL {
  min-height: 100vh;
}
</style>