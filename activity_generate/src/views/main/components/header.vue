<!--
 * @Author: your name
 * @Date: 2020-02-22 12:51:09
 * @LastEditTime: 2020-03-20 14:28:18
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: /activity_generate/src/components/header/index.vue
 -->
<template>
  <div class="header_con">
    <div class="header_back"></div>
    <div class="header">
      <div @click="gotoHome" class="left_header">
        <img class="left_logo" src="@/assets/logo.png" alt />
        <span>易动</span>
      </div>
      <!-- 左侧为操作栏 -->
      <div class="right_header">
        <a-button @click="saveObject(3)" type="primary" icon="book" class="right_header_item">保存为模板</a-button>
        <a-button @click="saveObject(2)" type="primary" icon="cloud" class="right_header_item">发布</a-button>
        <a-button
          @click="saveObject(1)"
          type="primary"
          icon="qrcode"
          class="right_header_item"
        >发布并预览</a-button>
      </div>
    </div>
    <!-- 发布预览框 -->
    <upload-modal ref="uploadModal" :objUrl="objUrl" />
    <!-- 发布模板 -->
    <set-template ref="setTemplate"></set-template>
    <!-- 权限校验 -->
    <auth-modal ref="authModal" @authSuccess="authSuccess"></auth-modal>
  </div>
</template>

<script>
// 主页面头部组件
import uploadModal from "./uploadModal";
import setTemplate from "./setTemplate";
import authModal from "@/components/authModal/index";
import { mobileUrl } from "@/config/index";
import html2canvas from "html2canvas";
import { base64ToBlob, BlobToImgFile } from "@/utils/index";
import { upTitleImg } from "@/api/index";

export default {
  components: {
    uploadModal,
    setTemplate,
    authModal
  },
  data() {
    return {
      objUrl: "" // 当前项目的url: value
    };
  },
  computed: {
    objectAuth() {
      return this.$store.state.core.objectAuth;
    },
    parentId() {
      return this.$store.state.core.parentId;
    }
  },
  methods: {
    gotoHome() {
      this.$router.push({ name: "home" });
    },
    // 获取缩略图
    getThumbnail() {
      this.$showLoading();
      // 保存之前取消选中 取消背景线的显示
      this.$store.commit("setting/toggle_backgroundLine");
      this.$store.commit("core/clear_template");
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          html2canvas(document.querySelector(".core"), {
            async: true,
            useCORS: true,
            scale: 1
          })
            .then(canvas => {
              let dataURL = canvas.toDataURL("image/jpeg");
              const data = new FormData();
              data.append(
                "image",
                BlobToImgFile(base64ToBlob(dataURL), "image/jpeg")
              );
              resolve(upTitleImg(data));
            })
            .catch(err => {
              reject(err);
            })
            .finally(() => {
              this.$hideLoading();
              this.$store.commit("setting/toggle_backgroundLine");
            });
        }, 1);
      });
    },
    // 通过效验
    authSuccess(data) {
      this.uploadObject(data.type, data.password);
    },
    // 保存项目
    saveObject(type) {
      if (type == 3) {
        // 保存模板
        this.getThumbnail().then(res => {
          this.$refs.setTemplate.openModal(res.data.data.data);
        });
      } else {
        if (this.objectAuth) {
          this.$refs.authModal.open({
            _id: this.parentId,
            type
          });
          return false;
        } else {
          this.uploadObject(type);
        }
      }
    },
    uploadObject(type, pass = "") {
      console.log(pass);
      this.getThumbnail().then(res => {
        // 保存当前页面的配置
        return this.$store
          .dispatch("core/saveObject", {
            titlePage: res.data.data.data,
            pass
          })
          .then(data => {
            console.log(data);
            this.$hideLoading();
            if (data.data.code == 200) {
              if (type == 1) {
                this.objUrl = mobileUrl + data.data.data;
                this.$refs["uploadModal"].openModal();
              } else {
                this.$message.success("发布成功");
              }
            } else {
              this.$message.error(data.data.data);
            }
          });
      });
    }
  }
};
</script>

<style lang="less" scoped>
.header_con {
  z-index: 2;
  min-height: 50px;
  height: 6%;
  background-color: #fafafa;
  .header_back {
    height: 100%;
    width: 100%;
  }
  .header {
    z-index: 1;
    position: fixed;
    top: 0px;
    left: 0px;
    right: 0px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 6%;
    min-height: 50px;
    background-color: rgb(251, 251, 251);
    // background-color: #001529;
    // color: white;
    box-shadow: 0 1px 4px rgba(0, 21, 41, 0.08);
    .left_header {
      cursor: pointer;
      margin-left: 20px;
      display: flex;
      align-items: center;
      .left_logo {
        width: 30px;
      }
    }
    .right_header {
      margin-right: 20px;
      .right_header_item {
        margin-right: 20px;
      }
    }
  }
}
</style>

<style lang="less">
</style>