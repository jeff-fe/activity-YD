<template>
  <div class="index_right">
    <!-- 侧边快捷操作栏 -->
    <div class="right_setting">
      <div
        class="setting_item"
        v-for="(item, index) in coreInfo"
        :key="index"
        @click="userSetting(index)"
      >
        <div v-if="item.text != '背景线'" :class="item.click ? 'item' : 'test item'">
          <img class="settion_item_icon" :src="item.icon" alt />
          <span class="settion_item_text">{{ item.text }}</span>
        </div>
        <div v-if="item.text == '背景线'" :class="item.click ? 'item' : 'test item'">
          <img v-if="backgroundLine" class="settion_item_icon" :src="item.openIcon" alt />
          <img v-if="!backgroundLine" class="settion_item_icon" :src="item.icon" alt />
          <span class="settion_item_text">{{ item.text }}</span>
        </div>
      </div>
      <p class="scale">{{ coreScale }}</p>
    </div>
    <!-- 组件设置 -->
    <a-tabs defaultActiveKey="1" @change="callback">
      <a-tab-pane tab="属性" key="1">
        <attributes-page />
      </a-tab-pane>
      <a-tab-pane tab="数据" key="2">
        <activedata-page />
      </a-tab-pane>
      <a-tab-pane tab="动画" key="3">
        <animation-page />
      </a-tab-pane>
    </a-tabs>
  </div>
</template>

<script>
import attributesPage from "./components/attributes";
import activedataPage from "./components/activedata";
import animationPage from "./components/animation";
import { baseComplate } from "@/utils/baseReact";
import { cloneDeep } from "lodash";
export default {
  components: {
    attributesPage,
    activedataPage,
    animationPage
  },
  data() {
    return {};
  },
  computed: {
    coreScale() {
      return Number((this.$store.state.setting.scale * 100).toFixed(1)) + "%";
    },
    backgroundLine() {
      return this.$store.state.setting.backgroundLine;
    },
    coreInfo() {
      return this.$store.state.setting.coreinfo;
    },
    activeTemplate() {
      return this.$store.state.core.activeTemplate;
    },
    template() {
      return this.$store.state.core.template;
    },
    copyTemplate() {
      return this.$store.state.setting.copyTemplate;
    }
  },
  watch: {
    activeTemplate() {
      if (this.activeTemplate.length > 0) {
        this.$store.commit("setting/set_coreinfoItem", {
          index: 2,
          status: true
        });
        this.$store.commit("setting/set_coreinfoItem", {
          index: 4,
          status: true
        });

        this.$store.commit("setting/set_coreinfoItem", {
          index: 6,
          status: true
        });
        this.$store.commit("setting/set_coreinfoItem", {
          index: 7,
          status: true
        });
      } else {
        this.$store.commit("setting/set_coreinfoItem", {
          index: 2,
          status: false
        });
        this.$store.commit("setting/set_coreinfoItem", {
          index: 4,
          status: false
        });
        this.$store.commit("setting/set_coreinfoItem", {
          index: 6,
          status: false
        });
        this.$store.commit("setting/set_coreinfoItem", {
          index: 7,
          status: false
        });
      }
    },
    copyTemplate() {
      console.log(this.copyTemplate);

      if (this.copyTemplate.length) {
        this.$store.commit("setting/set_coreinfoItem", {
          index: 3,
          status: true
        });
      }
    }
  },
  methods: {
    callback() {
      // 暂无右侧切换
    },
    //
    userSetting(index) {
      if (index == 0) {
        // 撤销
        this.$emit("coreSetting", "cancel");
      } else if (index == 1) {
        // 反撤销
        this.$emit("coreSetting", "uncancel");
      } else if (index == 2) {
        let copyList = [];
        this.activeTemplate.map(activityId => {
          copyList.push(
            cloneDeep(this.template.filter(e => e.activityId == activityId))[0]
          );
        });
        this.$store.commit("setting/set_copy", copyList);
        this.$message.success("已复制到粘贴板");
      } else if (index == 3) {
        this.copyTemplate.map(data => {
          this.$store.commit(
            "core/set_tempLate",
            cloneDeep(baseComplate(this.$store.state.core, data))
          );
        });
      } else if (index == 4) {
        this.$store.commit("core/deleteActiveComplate");
      } else if (index == 5) {
        this.$store.commit("setting/toggle_backgroundLine");
      } else if (index == 6) {
        this.$store.commit("core/update_CompZindex", 1);
      } else if (index == 7) {
        this.$store.commit("core/update_CompZindex", -1);
      } else if (index == 8) {
        this.$store.commit("setting/set_scale", 0.1);
      } else if (index == 9) {
        this.$store.commit("setting/set_scale", -0.1);
      }
    }
  }
};
</script>

<style lang="less" scoped>
.index_right {
  position: relative;
  background-color: #fafafa;
  width: 360px;
  height: 100%;
  display: flex;
  .right_setting {
    width: 45px;
    border-right: 1px solid #f6f6f6;
    .setting_item {
      width: 100%;
      height: 50px;
      user-select: none;
      cursor: pointer;
      text-align: center;
      display: flex;
      align-items: center;
      justify-content: center;
      &:hover {
        background: #f8f8f8;
      }
      .item {
        width: 325px;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        .settion_item_icon {
          width: 15px;
        }
        .settion_item_text {
          font-size: 10px;
          line-height: 18px;
        }
      }
      .test {
        opacity: 0.2;
        cursor: not-allowed;
      }
    }
    .scale {
      text-align: center;
    }
  }
}
</style>