<template>
  <div class="post-twitter-box">
    <ul class="tab-list">
      <li class="tab-item current">发表推文</li>
    </ul>
    <div class="twitter-box">
      <textarea
        v-model="content"
        @input="onInput"
        @keydown.ctrl.enter="doSubmit"
        @keydown.meta.enter="doSubmit"
        placeholder="有什么新鲜事想告诉大家"
        class="title-input"
      />
      <p class="words-number">{{ wordCount }}/{{ maxWordCount }}字</p>
      <div class="box-footer">
        <div class="bui-left">
          <span @click="showUploader = !showUploader" class="action-btn">
            <i class="iconfont icon-image" />
            <span>图片</span>
          </span>
          <!--
          <span class="action-btn">
            <i class="iconfont icon-emoji" />
            <span>表情</span>
          </span>
          -->
        </div>
        <div class="bui-right">
          <span class="msg-tip">{{ message }}</span>
          <a
            :class="{ active: hasContent }"
            @click="doSubmit"
            class="upload-publish"
            >发布</a
          >
        </div>
      </div>

      <div v-show="showUploader" class="uploader-popup">
        <div class="imgUploadBox">
          <p class="uploader-title">本地上传</p>
          <p class="uploader-meta">
            共 {{ imageCount }} 张，还能上传 {{ maxImageCount }} 张
          </p>
          <i
            @click="showUploader = false"
            class="close-popup iconfont icon-close"
          />
          <div class="upload-box">
            <form style="display: none;">
              <input
                ref="imageInput"
                @change="handleImageUploadChange"
                type="file"
                accept="image/*"
                multiple="multiple"
              />
            </form>
            <ul class="upload-img-list">
              <li v-for="image in images" :key="image" class="upload-img-item">
                <img :src="image" />
              </li>
              <li
                v-if="imageCount < maxImageCount"
                @click="handleImageUploadClick"
                class="upload-img-item upload-img-add"
              >
                <i class="iconfont icon-add" />
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    nodeId: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      content: '',
      images: [
        // 'https://file.mlog.club/images/2020/02/27/0aadf3d7c46dba756f4e228e8e8f8ed6.jpg?id=1'
        // 'https://file.mlog.club/images/2020/02/28/6819d3e0afb535594fb55c1108e1ad37.jpg'
      ],
      message: '',
      maxWordCount: 2000,
      showUploader: false,
      maxImageCount: 9
    }
  },
  computed: {
    hasContent() {
      return this.content && this.content.length > 0
    },
    wordCount() {
      return this.content ? this.content.length : 0
    },
    imageCount() {
      return this.images ? this.images.length : 0
    },
    user() {
      return this.$store.state.user.current
    }
  },
  methods: {
    onInput() {},
    async doSubmit() {
      if (!this.user) {
        this.message = '发表失败，请登录后重试'
        return
      }
      if (!this.hasContent) {
        this.message = '发表失败，请输入内容'
        return
      }
      this.showUploader = false // 关闭图片上传框
      try {
        const ret = await this.$axios.post('/api/topic/create', {
          type: 1,
          nodeId: this.nodeId,
          title: this.content,
          imageList: JSON.stringify(this.images)
        })
        this.content = ''
        this.message = ''
        this.$emit('created', ret)
        this.$toast.success('发布成功')
      } catch (e) {
        this.message = e.message || e
      }
    },
    handleImageUploadClick() {
      this.$refs.imageInput.click()
    },
    async handleImageUploadChange(ev) {
      const files = ev.target.files
      if (!files) return

      await this.uploadFiles(files)
    },
    async uploadFiles(files) {
      if (files.length === 0) {
        return
      }

      if (this.imageCount + files.length > this.maxImageCount) {
        this.message = '图片数量超过上限'
        return
      }

      for (let i = 0; i < files.length; i++) {
        await this.upload(files[i])
      }
    },
    async upload(file) {
      this.$refs.imageInput.value = null
      const formData = new FormData()
      formData.append('image', file, file.name)

      try {
        const ret = await this.$axios.post('/api/upload', formData)
        this.images.push(ret.url)
      } catch (e) {
        this.message = e.message || e
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.post-twitter-box {
  position: relative;
  border: 1px solid #e8e8e8;
  width: 100%;
  margin: 10px 0;

  .tab-list {
    height: 44px;
    border-bottom: 1px solid #e8e8e8;
    display: block;
    zoom: 1;

    .tab-item {
      margin-left: 19px;
      font-size: 15px;
      color: #222;
      line-height: 42px;
      border-bottom: 2px solid transparent;
      cursor: pointer;
      float: left;

      &.current {
        border-bottom-color: #ed4040;
        color: #222;
      }
    }
  }

  .twitter-box {
    padding: 0;
    margin: 0;
    box-sizing: border-box;
    position: relative;

    .title-input {
      width: 100%;
      height: 100px;
      display: block;
      font-size: 14px;
      line-height: 1.4;
      padding: 13px 19px;
      border: 0;
      outline: 0;
      resize: none;
      overflow: auto;
      background-color: #f4f5f6;
      transition: all 0.5s;
      animation-duration: 0.8s;
      animation-fill-mode: both;
    }

    .words-number {
      position: absolute;
      z-index: 3;
      bottom: 50px;
      right: 10px;
      display: inline-block;
      background-color: rgba(0, 0, 0, 0.5);
      border-radius: 50px;
      padding: 0 8px;
      color: #fff;
      font-size: 13px;
    }

    .box-footer {
      border-top: 1px solid #e8e8e8;
      height: 40px;
      display: block;
      zoom: 1;

      .bui-left {
        float: left;
        user-select: none;

        .action-btn {
          color: #222;
          font-size: 14px;
          line-height: 39px;
          display: inline-block;
          vertical-align: middle;
          margin: 0 0 0 20px;
          cursor: pointer;
          user-select: none;

          .iconfont {
            font-size: 20px;
            color: #ed4040;
            top: 2px;
            position: relative;
          }
        }
      }

      .bui-right {
        float: right;
        text-align: right;
        user-select: none;

        .msg-tip {
          color: #ed4040;
          font-size: 12px;
          margin-right: 10px;
        }

        .upload-publish {
          display: inline-block;
          width: 120px;
          line-height: 40px;
          text-align: center;
          font-size: 14px;
          background-color: #ed4040;
          color: #fff;
          opacity: 0.6;
          user-select: none;

          &.active {
            opacity: 1;
          }
        }
      }
    }

    .uploader-popup {
      position: absolute;
      bottom: -245px;
      left: -1px;
      width: 420px;
      height: 246px;
      background-color: #fff;
      border: 1px solid #e8e8e8;
      box-shadow: 0 2px 15px rgba(0, 0, 0, 0.12);
      border-radius: 5px;
      z-index: 300;

      &:after {
        content: '';
        width: 0;
        height: 0;
        border: 6px solid transparent;
        border-bottom-color: #fff;
        position: absolute;
        top: -12px;
        left: 36px;
      }

      .imgUploadBox {
        padding: 12px;

        .uploader-title {
          margin-bottom: 5px;
          font-size: 14px;
          color: #222;
        }

        .uploader-meta {
          color: #999;
          font-size: 14px;
          margin-bottom: 10px;
        }

        .close-popup {
          position: absolute;
          top: 8px;
          right: 6px;
          color: #cacaca;
          font-size: 18px;
          font-weight: 700;
          cursor: pointer;
        }

        .upload-box {
          padding: 0;
          margin: 0;
          box-sizing: border-box;
          position: relative;
          display: block;
          zoom: 1;

          .upload-img-list {
            font-size: 0;
            margin-right: -8px;

            .upload-img-add {
              background-color: #fff !important;
              text-align: center;

              i.icon-add {
                font-size: 30px;
                color: rgb(221, 221, 221);
              }
            }

            .upload-img-item {
              cursor: pointer;
              border: 2px dashed #ddd;
              line-height: 72px;
              text-align: center;

              display: inline-block;
              vertical-align: middle;
              width: 72px;
              height: 72px;
              margin: 0 8px 8px 0;
              background-color: #e8e8e8;
              background-size: 32px 32px;
              background-position: 50%;
              background-repeat: no-repeat;
              overflow: hidden;
              position: relative;
            }
          }
        }
      }
    }
  }
}
</style>
