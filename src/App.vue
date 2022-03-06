<template>
  <div id="app">
    <div class="title">H4Pay 관리 앱 배포</div>
    <div class="container">
      <b-field label="헌재 버전" horizontal>
        {{ currVersion }} ( {{ currVersionCode }} )
      </b-field>
      <b-field label="APK 파일 업로드" horizontal>
        <b-upload
          v-model="dropFile"
          drag-drop
          accept=".apk"
          @input="fetchApkInfo"
        >
          <section class="section">
            <div class="content has-text-centered">
              <p>
                <b-icon icon="upload" size="is-large"> </b-icon>
              </p>
              <p v-if="typeof dropFile == 'function'">
                파일을 선택하거나 드래그하세요
              </p>
              <div v-else>
                {{ dropFile.name }}
              </div>
            </div>
          </section>
        </b-upload>
      </b-field>
      <p v-if="isUploaded == true">파일 파싱 중...</p>
      <fieldset v-if="nextVersion != null">
        <b-field label="업데이트 대상 버전" horizontal>
          {{ nextVersion }} ( {{ nextVersionCode }} )
        </b-field>
        <b-field label="변경사항" horizontal>
          <b-input type="textarea" v-model="changes" />
        </b-field>
        <b-field>
          <b-button rounded style="float: right" @click="deploy">배포</b-button>
        </b-field>
      </fieldset>
      <b-modal
        has-modal-card
        v-model="uploadModalActive"
        title="업로드"
        aria-role="dialog"
        aria-label="Example Modal"
        aria-modal
      >
        <div class="modal-card" style="width: auto">
          <header class="modal-card-head">
            <p class="modal-card-title">파일 업로드</p>
          </header>
          <section class="modal-card-body">
            <p style="margin-bottom: 3%">
              파일 {{ dropFile ? dropFile.name : "" }}
            </p>
            <p v-if="!uploaded">업로드 현황: {{ percent }} %</p>
            <p v-if="uploaded">업로드 완료</p>
            <b-progress :value="percent" show-value style="width: 100%">
              {{ percent }}
            </b-progress>
          </section>
        </div>
      </b-modal>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      dropFile: File,
      currVersion: null,
      currVersionCode: null,
      nextVersion: null,
      nextVersionCode: null,
      changes: "",
      uploaded: false,
      uploadModalActive: false,
      percent: 0,
    };
  },
  created() {
    this.$axios
      .get(`${process.env.VUE_APP_API_URL}/versions`)
      .then((res) => {
        this.currVersion = parseFloat(res.data.result.version);
        this.currVersionCode = parseInt(res.data.result.versionCode);
      })
      .catch((err) => {
        alert(`현재 버전을 가져올 수 없습니다: ${err.message}`);
      });
  },
  methods: {
    uploadApk(data) {
      let config = {
        onUploadProgress: (progressEvent) => {
          var percentCompleted = Math.round(
            (progressEvent.loaded * 100) / progressEvent.total
          );

          // execute the callback
          this.percent = percentCompleted;
          return percentCompleted;
        },
        headers: {
          "Content-Type": "multipart/form-data",
        },
      };
      return this.$axios
        .post(process.env.VUE_APP_API_URL + "/versions/update", data, config)
        .then((x) => x.request.response)
        .catch((error) => error);
    },
    async fetchApkInfo(file) {
      const parser = await new this.$apkReader(file);
      parser
        .parse()
        .then((res) => {
          this.nextVersion = res.versionName;
          this.nextVersionCode = res.versionCode;
        })
        .catch((err) => {
          this.nextVersion = null;
          this.nextVersionCode = null;
          this.uploaded = false;
          alert(`APK 파싱에 실패했습니다: ${err}`);
        });
    },
    deploy() {
      if (!this.nextVersionState) {
        alert("버전 정보가 올바르지 않습니다!");
        return;
      } else if (!this.changeState) {
        alert("변경사항이 올바르지 않습니다!");
        return;
      }

      let form = new FormData();
      form.append("nextVersion", this.nextVersion);
      form.append("nextVersionCode", this.nextVersionCode);
      form.append("changes", this.changes);
      form.append("apk", this.dropFile);
      this.uploadModalActive = true;
      this.uploadApk(form, this.onProgress)
        .then((x) => {
          console.log(x);
          const res = JSON.parse(x);
          if (res.status == true) {
            this.uploaded = true;
            // Check file downloadable
            this.uploadModalActive = false;
            alert("파일 업로드에 성공했습니다!");
            window.location.reload()
          } else if (res.status == false) {
            const error = new Error("file upload failed");
            this.$Sentry.captureException(error);
            alert("파일 업로드에 실패했습니다.");
          }
        })
        .catch((error) => {
          this.$Sentry.captureException(error);
          alert("파일 업로드에 실패했습니다.");
          console.log(error);
        });
    },
  },
  computed: {
    nextVersionState() {
      return (
        this.nextVersion != null &&
        this.nextVersion.toString().indexOf(".") != -1 &&
        this.nextVersion != ""
      );
    },
    changeState() {
      return (
        this.changes != null && this.changes != "" && this.changes.length != 0
      );
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100vh;
  width: 100vw;
  padding: 10vw;
}
.container {
  border: 1px solid #ccc !important;
  border-radius: 15px;
  padding: 20px;
  padding-bottom: 6vh;
}

.upload {
  width: 100% !important;
}
.upload-draggable {
  width: 100% !important;
}
</style>
