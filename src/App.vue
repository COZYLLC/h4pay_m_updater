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
      <p v-if="isUploaded == true">
        파일 파싱 중...
      </p>
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
      isUploaded: false,
    };
  },
  created() {
    this.$axios
      .get(`${process.env.VUE_APP_API_URL}/version`)
      .then((res) => {
        this.currVersion = parseFloat(res.data.result.version);
        this.currVersionCode = parseInt(res.data.result.versionCode);
      })
      .catch((err) => {
        alert(`현재 버전을 가져올 수 없습니다: ${err.message}`);
      });
  },
  methods: {
    async fetchApkInfo(file) {
      this.isUploaded = true;
      const parser = await new this.$apkReader(file);
      parser.parse().then(res => {
        this.nextVersion = res.versionName;
        this.nextVersionCode = res.versionCode;
        this.isUploaded = false;
      }).catch(err => {
        this.nextVersion = null;
        this.nextVersionCode = null;
        this.isUploaded = false;
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

      this.$axios
        .post(`${process.env.VUE_APP_API_URL}/version/update`, form)
        .then((res) => {
          if (res.data.status) {
            this.$axios
              .get(res.data.testUrl)
              .then((result) => {
                if (result.status == 200) {
                  alert("배포가 완료되었습니다!");
                }
              })
              .catch((err) => {
                alert(
                  `업로드가 제대로 이뤄지지 않은 것 같아요.\n${err.message}`
                );
              });
          }
        })
        .catch((err) => {
          alert(`업로드가 제대로 이뤄지지 않은 것 같아요.\n${err.message}`);
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
